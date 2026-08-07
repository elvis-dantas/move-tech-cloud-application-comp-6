C4Container
    title Diagrama de Contêineres (C2) - Arquitetura Atual

    Person(user, "Usuário Final", "Usuário da aplicação")

    System_Boundary(k3s_mgc, "Cluster K3s (Magalu Cloud - MGC)") {
        Container(loadbalancer, "Service LoadBalancer", "Kubernetes Service", "IP externo (Porta 80). Encaminha tráfego para os pods.")
        
        Container(cloud_app, "cloud-application (2 pods)", "FastAPI (Python)", "Processa requisições de negócios e expõe /orders, /items, /health, /stats, /metrics. Escala para 300 req/s.")
        
        Container(db_secret, "db-secret", "Kubernetes Secret", "Armazena credenciais de acesso ao PostgreSQL.")

        System_Boundary(monitoring_ns, "namespace: monitoring") {
            Container(prometheus, "Prometheus", "ServiceMonitor", "Scrapes métricas do /metrics via ServiceMonitor. Monitora latência P95 < 500ms.")
            Container(grafana, "Grafana", "Dashboard", "Visualiza disponibilidade (Erros 5xx) e uptime das probes.")
        }
    }

    SystemDb_Ext(postgres_managed, "PostgreSQL Gerenciado", "PostgreSQL (DBaaS)", "Fora do cluster. Base de dados principal.", $tags="external_system")

    Rel(user, loadbalancer, "Acessa via HTTP", "port 80")
    Rel(loadbalancer, cloud_app, "Encaminha tráfego", "port 8000")
    
    Rel_R(cloud_app, db_secret, "Lê credenciais")
    Rel(cloud_app, postgres_managed, "Lê/Grava dados", "SQL/TCP 5432")

    Rel_U(prometheus, cloud_app, "Coleta métricas (/metrics)", "HTTP")
    Rel_U(grafana, prometheus, "Lê métricas", "HTTP")

    UpdateElementStyle(postgres_managed, $bgColor="#333", $textColor="#fff", $borderColor="#fff")