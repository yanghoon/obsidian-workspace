
### Trino
* [Deploying Trino](https://trino.io/docs/current/installation/deployment.html)
* AddProperty("Java Version", "23 (Temurin)")
* AddProperty("Java Options", "-Xmx54G (over 32GB, 70~85%)")
* Coordinator
	* AddProperty("http-server.http.port", "8080")
	* AddProperty("discovery.uri", "http://example.net:8080")
	* AddProperty("node.id", "coordinator")
	* AddProperty("node.data-dir", "${WORKSPACE}/data/coordinator")
* Worker
	* AddProperty("Java Version", "23 (Temurin)")
	* AddProperty("Java Options", "-Xmx54G (over 32GB, 70~85%)")
	* AddProperty("http-server.http.port", "8080")
	* AddProperty("discovery.uri", "http://example.net:8080")
	* AddProperty("node.id", "worker**")
	* AddProperty("node.data-dir", "${WORKSPACE}/data/worker")
* H/W Spec
	* `Trino 475 클러스터 HW 사양 추천 (데이터 규모: 10GB 미만/일)`
	* 코디네이터 : 4 Core / 16GB RAM / 100GB SSD
	* 작업자 : 8 Core / 32GB RAM / 200GB SSD (2~3대)
	* 네트워크 1Gbps 이상, SSD 사용 권장
### Flink
* H/W Spec
	* `Flink 1.20 클러스터 HW 사양 추천 (데이터 규모: 10GB 미만/일)`
	* JobManager : 4 Core / 16GB RAM / 100GB SSD
	* TaskManager : 8 Core / 32GB RAM / 200GB SSD (2~3대)

```plantuml
@startuml

' !include <C4/C4_Container>
!include <C4/C4_Deployment>

Deployment_Node(vm_local, "Local") {
	Person_Ext(developer, "Developer")
	Container_Ext(local_trino, "Trino", "Docker Compose")
	Container_Ext(local_ide, "IDE", "VSCode")
	Container_Ext(local_gui, "GUI Tools", "DBeaver")

	Rel(developer, local_trino, "")
	Rel(developer, local_ide, "")
	Rel(developer, local_gui, "")
}

System_Boundary(flink, "Flink Cluster") {
	Deployment_Node(vm_flink_jobs, "flink-jobs-**\tx1", "RHEL8") {
		Container(flink_job, "Job Manager", "Flink v1.20.1")
	}
	Deployment_Node(vm_flink_tasks, "flink-workers-***\tx3", "RHEL8") {
		Container(flink_task, "Task Manager", "Flink v1.20.1")
	}

	Rel(flink_job, flink_task, "", "Flink Task")
}

System_Boundary(trino, "Trino Cluster") {
	Deployment_Node(vm_trino_coordinator, "trino-cord-**\tx1", "RHEL8") {
		Container(trino_coordinator, "Coodinator", "Trino 475")
		Container(trino_iceberg_catalog_rest, "Iceberg Catalog", "REST")
	}
	Deployment_Node(vm_trino_workers, "trino-workers-**\tx3", "RHEL8") {
		Container(trino_worker, "Worker", "Trino 475")
	}

	Rel(trino_coordinator, trino_worker, "")
}

ContainerDb_Ext(lake, "Data Lake", "Object Storage")

'' Rel
Rel(local_trino, lake, "", "Iceberg SDK")
Rel(local_ide, lake, "", "Iceberg SDK")
Rel(local_ide, trino_coordinator, "", "JDBC")
Rel(local_gui, trino_coordinator, "", "JDBC")

Rel(developer, flink_job, "Upload Flink Job (jar)", "REST / Browser")

Rel(flink_task, lake, "", "S3 API")

Rel(trino_worker, lake, "", "S3 API")

@enduml
```
