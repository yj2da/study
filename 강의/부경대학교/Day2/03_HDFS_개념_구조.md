# 03. HDFS 개념 및 구조

## HDFS (Hadoop Distributed File System)란?

- **역할**: 대용량 데이터를 **여러 서버에 블록 단위로 나누어 저장**하는 분산 파일 시스템입니다.
- **배경**: Google의 **GFS(Google File System)** 논문에 기반한 Apache Hadoop의 저장소입니다. 한 대에 다 담기 어려운 데이터를 여러 노드에 분산해 저장·복제합니다.

---

## HDFS 아키텍처

HDFS는 **NameNode(마스터)** 와 **DataNode(워커)** 로 이루어진 마스터–슬레이브 구조입니다. 
클라이언트는 메타데이터 작업은 NameNode에, 실제 데이터 읽기·쓰기는 DataNode에 직접 요청합니다.

![HDFS 아키텍처: NameNode, DataNode, 클라이언트 읽기/쓰기, 복제](img/hdfsarchitecture.png)

- **NameNode**: 클라이언트의 **메타데이터 요청(Metadata ops)** 을 받아, 파일 경로·블록 위치·복제 수(예: `/home/foo/data`, replicas 3) 등을 관리합니다. 실제 파일 데이터는 저장하지 않고, **블록 작업(Block ops)** 을 DataNode에 지시합니다.
- **DataNode**: **블록(Blocks)** 단위로 실제 데이터를 저장합니다. 그림처럼 여러 DataNode가 **Rack 1**, **Rack 2** 등 랙에 나뉘어 있고, **복제(Replication)** 는 랙을 넘어 한 Datanode에서 다른 랙의 Datanode로 블록이 복사됩니다.
- **클라이언트 읽기**: NameNode에서 블록 위치를 받은 뒤, 해당 **DataNode에서 직접** 데이터를 읽습니다.
- **클라이언트 쓰기**: 한 DataNode(예: Rack 1)에 쓰면, 그 DataNode가 다른 랙의 DataNode로 복제를 이어가며 **파이프라인** 형태로 복제가 진행됩니다.

### NameNode

- **메타데이터 관리**: 파일 이름, 디렉토리 구조, 각 파일이 **어떤 블록들로 나뉘어 어떤 DataNode에 있는지** 정보를 가집니다.
- **클라이언트 요청 처리**: 파일 읽기/쓰기 요청이 오면, 해당 블록의 위치를 알려 주고 조정합니다.
- **단일 NameNode**: 기본 구성에서는 NameNode가 한 대라, 고가용성(HA) 구성을 쓰지 않으면 NameNode가 단일 장애점이 됩니다.

### DataNode

- **실제 데이터 저장**: 파일이 **블록(block)** 단위로 쪼개져 DataNode들의 디스크에 저장됩니다.
- **기본 블록 크기**: 128MB (설정으로 변경 가능). 큰 파일은 여러 블록으로 나뉘어 여러 DataNode에 흩어집니다.
- **복제(Replication)**: 같은 블록을 기본적으로 **3벌**로 복제해, 일부 노드 장애 시에도 데이터를 잃지 않도록 합니다.

> 💡 **왜 복제 수(Replication Factor)를 3으로 할까?**  
> 
> **홀수(3)를 권장하는 이유:**
> - **과반수(Quorum) 유지**: 3개 중 1개가 죽어도 2개가 남아 과반수(2/3)가 유지되어 읽기/쓰기가 가능합니다
> - **짝수(2)의 문제**: 1개만 죽어도 50%가 되어 투표 기반 구조에서 과반수가 깨질 수 있습니다
> 
> **효율성 고려:**
> - 너무 많으면(5+) 저장 공간 낭비
> - 너무 적으면(1-2) 데이터 유실 위험 높음
> - **3이 안정성과 효율성의 균형점**
> 
> 실무에서는 보통 2~5개 내외로 설정합니다. JournalNode, ZooKeeper 같은 Hadoop 생태계 컴포넌트들도 3, 5, 7 등 홀수 구성을 권장합니다.

![HDFS DataNode: 블록 저장 및 복제 파이프라인](img/hdfsdatanodes.png)

- 위 그림은 DataNode가 **블록을 어떻게 저장하고**, 쓰기 시 **복제 파이프라인**으로 다른 DataNode에 전달하는 흐름을 나타냅니다. 클라이언트가 한 DataNode에 블록을 쓰면, 그 DataNode가 다음 복제본 DataNode로, 그 다음이 또 다음 DataNode로 차례로 전달하는 방식입니다.

### Rack Awareness

- **개념**: 복제본을 **서버 랙(rack)** 을 고려해 배치합니다. 같은 랙만 쓰면 랙 전체 장애 시 복제본을 잃을 수 있으므로, 다른 랙에도 복제본을 두는 전략을 씁니다.

---

## 복제(Replication) 전략

- **3-way replication**: 기본 설정. 원본 1 + 복제 2 = 총 3벌. 하나의 DataNode나 디스크가 죽어도 다른 복제본으로 읽기·쓰기가 가능합니다.
- **장애 복구**: DataNode가 죽으면 NameNode가 부족한 복제본을 다른 노드에 다시 만들어 복제 수를 유지합니다.

---

## 핵심 개념 정리

- **HDFS**: 대용량 데이터를 블록 단위로 여러 서버에 분산 저장하는 분산 파일 시스템. GFS 논문 기반.
- **NameNode**: 메타데이터(파일·블록 위치) 관리. **DataNode**: 실제 블록 데이터 저장.
- **블록**: 기본 128MB. 복제(기본 3벌), Rack Awareness로 내결함성 확보.

---

## 참고

- [HDFS 공식 문서 (HdfsDesign)](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
- [Google GFS 논문](https://research.google/pubs/pub51/)
- [04_분산저장_아키텍처.md](04_분산저장_아키텍처.md) — HDFS 외 분산 저장 시스템(Ceph, MinIO, Cloud 등) 비교.

---

## 그림 출처

- **img/hdfsarchitecture.png**: HDFS 아키텍처(NameNode, DataNode, 클라이언트 읽기/쓰기, 랙 간 복제). 출처: [Apache Hadoop HdfsDesign](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html).
- **img/hdfsdatanodes.png**: HDFS DataNode 블록 저장 및 복제 파이프라인. 출처: [Apache Hadoop HdfsDesign](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html).
