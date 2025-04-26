*Trabalho de monitoramento de logs*

Este repositório contém a configuração para um pipeline de monitoramento de logs usando Docker Compose, Grafana Loki e Promtail.

📁 Estrutura de diretórios

.
├── docker-compose.yml       # Orquestração dos serviços
├── promtail/
│   ├── config.yaml         # Configuração do Promtail
│   └── position/
│       └── positions.yaml  # Arquivo de posições de leitura
└── logs/
    └── gerador-log/
        └── record.log      # Exemplo de logs gerados

📋 Descrição dos arquivos

**1. docker-compose.yml**

Define e conecta quatro serviços principais em uma rede Docker "loki":

gerador-logs:

Imagem: fabricioveronez/gerador-log

Mapeia a porta 8080 para 5000 e monta ./logs/gerador-log para persistência.

Gera logs que são colhidos pelo Promtail.

promtail:

Imagem: grafana/promtail:2.7.5

Depende dos serviços loki e gerador-logs.

Porta HTTP de scraping: 9080.

Volumes montados:

promtail/config/config.yaml → /etc/promtail/config.yml

/var/log → /logs/syslogs/

./logs/gerador-log → /logs/gerador-log

./promtail/position → /position

Responsável por coletar (scrape) e enviar logs para o Loki.

loki:

Imagem: grafana/loki:2.7.5

Porta de API: 3100.

Recebe requests de push de logs do Promtail.

grafana:

Imagem: grafana/grafana

Porta de interface: 3000.

Conecta-se ao Loki para visualização e dashboard de logs.

**2. config.yaml**

Arquivo de configuração principal do Promtail:

server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /position/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
    - localhost
    labels:
      job: varlogs
      __path__: /logs/syslogs/*log
  - targets:
    - localhost
    labels:
      job: gerador-log
      __path__: /logs/gerador-log/*log

server: Porta usada pelo Promtail para health-check.

positions: Armazena o offset de leitura para retomar logs onde parou.

clients: URL de destino para envio de logs (Loki).

scrape_configs: Define dois jobs:

varlogs: captura todos os arquivos .log dentro de /logs/syslogs/ (logs do sistema).

gerador-log: captura o arquivo record.log do serviço gerador.

**3. positions.yaml**

Armazena as últimas posições de leitura de cada arquivo de log para evitar reenvio de linhas já processadas:

positions:
  /logs/gerador-log/record.log: "319"
  /logs/syslogs/alternatives.log: "22399"
  /logs/syslogs/auth.log: "15943"
  /logs/syslogs/bootstrap.log: "118497"
  /logs/syslogs/dpkg.log: "379341"
  /logs/syslogs/faillog: "0"
  /logs/syslogs/fontconfig.log: "790"
  /logs/syslogs/kern.log: "51280"
  /logs/syslogs/lastlog: "0"
  /logs/syslogs/syslog: "298098"

Cada entrada mapeia o caminho de um arquivo à linha já consumida.

**4. record.log**

Exemplo de registros gerados pelo serviço gerador-logs:

2025-04-26 18:56:31,140 DEBUG    app          MainThread   Teste
2025-04-26 19:33:23,260 DEBUG    app          MainThread   Teste
2025-04-26 19:44:14,682 INFO     app          MainThread   Teste
2025-04-26 19:44:20,551 INFO     app          MainThread   Teste
2025-04-26 19:44:24,817 WARNING  app          MainThread   Teste

Formato de timestamp: YYYY-MM-DD HH:MM:SS,mmm

Nível de log: DEBUG, INFO, WARNING, etc.

▶️ *Como iniciar o ambiente*

Clone o repositório:

git clone https://github.com/neresfelip/trabalho-pos-monitoramento-logs
cd monitoramento-de-logs

Suba os serviços:

docker-compose up -d

Acesse as interfaces:

Grafana: http://localhost:3000

Loki API: http://localhost:3100

Promtail (health): http://localhost:9080

Gerador de Logs: http://localhost:8080

🛠️ *Observações*

Para zerar as posições e remontar todos os logs, apague o arquivo promtail/position/positions.yaml.

Ajuste o path em __path__ no config.yaml caso adicione novos diretórios de logs.

*Prints:*

![Grafana lendo os logs gerados pelo gerador de log](prints/print1.png)
![Grafana lendo os logs do varlogs do Linux](prints/print2.png)
![Targets de logs definidos pelo promtails](prints/print3.png)

Autor: Felipe Neres Ribeiro
Data: 26-04-2025