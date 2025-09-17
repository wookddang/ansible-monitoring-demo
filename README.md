# 시스템 아키텍처 (VMware 기반 Rocky Linux 9.0 클러스터)

```mermaid
flowchart LR
    classDef bigText fill:#E9F7EF,stroke:#2ECC71,stroke-width:1px,font-size:18px;
    subgraph DC["VMware 기반 Rocky Linux 9.0 노드"]
        A["사용자/브라우저"]
        
        subgraph tnode1["Web/Mon Node tnode1"]
            Nginx["Nginx :80 /healthz"]:::bigText
            Prom["Prometheus :9090 (Podman)"]:::bigText
            AM["Alertmanager :9093 (Podman)"]:::bigText
            GFN["Grafana :3000 (Podman)"]:::bigText
            NE1["node_exporter :9100"]:::bigText
        end

        subgraph tnode2["App Node tnode2"]
            Blue["demoapp-blue :5001 (systemd)"]
            Green["demoapp-green :5002 (systemd)"]
            NE2["node_exporter :9100"]
        end

        subgraph ansible_server["Control (ansible-server)"]
            ANS["Ansible Core 2.14.18<br/>Playbooks/Inventory"]
        end
    end

    %% 프론트 경로
    A -->|"HTTP :80"| Nginx
    Nginx -->|"upstream 색상 스위치"| Blue
    Nginx -.->|"upstream 대체"| Green

    %% 모니터링 경로
    Prom <-->|"scrape :9100"| NE1
    Prom <-->|"scrape :9100"| NE2
    Prom -. scrape .- Prom
    Prom -. scrape .- AM
    GFN -->|"Query"| Prom
    Prom -->|"Alerts"| AM

    %% 자동화 경로
    ANS -. SSH .-> Nginx
    ANS -. SSH .-> Blue
    ANS -. SSH .-> Green
    ANS -. SSH .-> NE1
    ANS -. SSH .-> NE2
    ANS -. SSH .-> Prom
    ANS -. SSH .-> AM
    ANS -. SSH .-> GFN

    %% 보안/베이스라인 라벨
    classDef secure fill:#E9F7EF,stroke:#2ECC71,stroke-width:1px,color:#1D8348;
    class Nginx,Blue,Green,Prom,AM,GFN,NE1,NE2 secure
