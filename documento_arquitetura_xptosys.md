# Projeto XPTO
#### Documento de Arquitetura - Sistema XPTOSys
*Desenvolvido por Rogerio Pacheco (Maio/25)*

##### 1. Objetivo
Implementar na empresa XPTO o sistema de fluxo de caixa XPTOSys em ambiente híbrido (on premise e cloud), garantindo alta disponibilidade, resiliência, segurança, escalabilidade e governança de custos.

##### 2. Considerações
Tendo por base os requisitos do projeto, foram definidas as premissas abaixo para o desenvolvimento da arquitetura do sistema XPTOSys:

- ***Transformação digital:*** A empresa XPTO encontra-se em fase de transformação digital com uma estratégia orientada à migração de seus workloads críticos para a nuvem. Dentre esses workloads críticos, o sistema XPTOSys destaca-se por ser o motivador do projeto em questão.
 
- ***Segurança:*** Visando a segurança da infraestrutura de ambos os ambientes produtivos, considerou-se que o sistema terá somente acesso privado a partir das lojas e matriz (escritório) sem nenhum tipo de exposição à internet. Todos os protocolos envolvidos nas comunicações e administração dos recursos de infraestrutura contam com criptografia e certificados digitais.

- ***Continuidade de negócios, alta disponibilidade e resiliência:*** Ambiente on premise de médio porte já existente que requer maior escalabilidade, resiliência e implementações de novos recursos devido ao crescimento da empresa. Os investimentos envolvidos nas melhorias acima mencionadas para o ambiente on premise tornam-se pouco atrativos quando comparados com aqueles envolvidos na infraestrutura em nuvem, considerando-se que a mesma possui maior capacidade de escalabilidade, resiliência e disponibilidade. 

Para reforçar a alta disponibilidade e performance do sistema XPTOSys e também preservar os investimentos feitos na estrutura on premise, a mesma será mantida como ambiente produtivo ativo juntamente com a nova infraestrutura em nuvem e também poderá funcionar como DR (Disaster Recovery). 

##### 3. Decisões de Arquitetura
###### 3.1. Cloud
No intuito de viabilizar a jornada de transformação digital da XPTO para a nuvem é necessário adotar um cloud provider capaz de ofertar uma plataforma intuitiva e que tenha compatibilidade com sistemas de mercado e ambientes híbridos.

A Azure é capaz de proporcionar as experiências acima mencionadas, além de possuir ampla oferta para mão de obra especializada, suporte técnico integral e diversos pontos de presença ao redor do mundo, o que a torna uma plataforma flexível e robusta o suficiente para atender aos requisitos do projeto.

##### 3.2. Topologia do sistema
O acesso ao sistema será feito de forma privativa por matriz e lojas sem nenhum tipo de exposição à internet.

O sistema está disposto em dois componentes: front end (conteúdo estático) feito em Angular e backend (regras de negócio) desenvolvido em .Net.

Na infra em nuvem o front end será executado em cluster kubernetes e o back end executado em Azure Functions. O uso da tecnologia .Net facilitará o desenvolvimento e as eventuais integrações requeridas para execução no Azure Functions.

No ambiente on premise o front end e back end da aplicação estão dispostos na mesma estrutura de servidores com uso das mesas tecnologias previstas na infra em nuvem.

A aplicação foi concebida para funcionar com 2 endereços de acesso ativos e produtivos, sendo sys1.xpto.corp e sys2.xpto.corp.

O endereço sys1.xpto.corp é responsável pelo acesso ao sistema a partir da infraestrutura em nuvem, enquanto o endereço sys2.xpto.corp é responsável pelo acesso ao sistema a partir do ambiente on premise.

Tendo por base que um dos requisitos de projeto é o alto desempenho dos serviços de Controle de Lançamentos e Consolidado Diário (componentes de back end do sistema) frente a um intenso volume de acessos, o uso da computação serverless (Azure Functions) foi considerado para esses serviços, pois um de seus benefícios principais é a execução em alto desempenho de aplicações que possuem grandes volumes de acesso com alocação de infraestrutura sob demanda por parte do cloud provider, ou seja, não é necessário alocar e pagar por uma infraestrutura dedicada e complexa em tempo integral para a execução desses serviços, visto que a infraestrutura do Azure Functions é fornecida pelo cloud provider com base no volume de utilização dos serviços em questão.

Por outro lado, o front end da aplicação é previsto para execução em kubernetes, o qual proporcionará um desempenho satisfatório para um componente de front end com uma infraestrutura sem complexidade a um custo-benefício atrativo quando comparado aos custos envolvidos no Azure Functions.

##### 3.3. Banco de dados e cache
Visando a integridade e sincronismo dos dados, bem como das transações efetuadas pelo sistema, o projeto contempla a replicação entre os bancos de dados existentes entre o ambiente on premise e a infra em nuvem.

O Azure SQL Managed Instances será implementado na estrutura em nuvem e o SQL Server montado em cluster foi considerado para uso no ambiente on premise. A replicação dos dados ocorrerá através do recurso Bidirectional Transactional Replication existente em ambos gerenciadores de banco de dados (SQL Server e Azure SQL Managed Instances).

A solução Microsoft SQL (Azure SQL Managed Instances e SQL Server) foi adotada tendo por base a facilidade de seu uso e eventuais integrações dentro da Azure, além de trazer como benefícios para o projeto a utilização do banco de dados na modalidade PaaS que dispensa a alocação e gestão de infraestrutura específica para essa finalidade.

O SQL Server configurado em cluster foi considerado no ambiente on premise para viabilizar a replicação de dados com o Azure SQL Managed Instances utilizando os recursos de infraestrutura já existentes.

Atendendo ao requisito de projeto referente ao alto desempenho das transações com baixa latência em seus tempos de respostas, foi adotado o uso do Redis montado em cluster no ambiente on premise e Azure Cache for Redis configurado na estrutura em nuvem como soluções de cache para armazenamento dos dados acessados com mais frequência pelo sistema e gerenciamento de sessões que ajudarão em sua estabilidade e consistência.

##### 3.4. Mensageria (Streaming)
Considerando que os serviços do sistema executados no Azure Functions processarão e transmitirão entre si fluxos de dados em tempo real que vão requerer desempenho otimizado e alto grau de confiabilidade, é proposto o uso do Azure Service Bus na estrutura em nuvem e Kafka no ambiente on premise.

O uso do Azure Service Bus no ambiente em nuvem trará como benefícios adicionais: facilidade de integração com Azure Functions e isenção da alocação e gestão de infraestrutura dedicada para essa finalidade, visto que esse serviço é ofertado na modalidade PaaS (Platform as a Service).

O uso do Kafka em cluster no ambiente on premise tem como benefícios: alta disponibilidade do serviço de mensageria e redução do custo total de propriedade (TCO), pois trata-se de uma solução open source consolidada no mercado que requer apenas os recursos de infraestrutura já existentes para execução.

##### 3.5. Conectividade
As lojas possuem conectividade redundante de operadoras distintas (Links MPLS) para acesso ao sistema através da infra em nuvem e também via ambiente on premise.

Os links MPLS atenderão aos requisitos do projeto, pois oferecem estabilidade, alto desempenho e proteção no tráfego.

A matriz da XPTO possui conectividade redundante de operadoras distintas (Links MPLS) para acesso aos ambientes em nuvem e on premise. Adicionamente possui dois links de internet para uso do Azure DevOps.

A estrutura on premise possui conectividade direta redundante (Express Route) com a infra em nuvem para replicação de dados, execução das rotinas de backup, acessos aos servidores e demais recursos de ambos os ambientes, além do acesso por parte das lojas ao sistema hospedado nessa estrutura.

##### 3.6. Segurança
O projeto contempla o uso de firewalls implementados em alta disponibilidade para a infra em nuvem, ambiente on premise e matriz da XPTO.

Os firewalls FortiGate adotados para o projeto são classificados como firewalls de próxima geração (NGFW) contemplando portanto não somente a inspeção de tráfego tradicional como também regras para controle de acesso ao sistema XPTOSys, balanceamento e redundância de links e configurações de roteamento e rede nos ambientes envolvidos. Adicionalmente às necessidades do projeto, esses firewalls poderão atuar na filtragem e controle do conteúdo a ser acessado via internet pelo escritório matriz.

Os protocolos utilizados na administração dos servidores e para acesso ao sistema contam com criptografia e certificados digitais.

Para acesso protegido aos servidores Linux é considerado o uso do protocolo SSH, o qual opera com par de chaves pública e privada, enquanto os servidores Windows utilizarão o protocolo RDP com criptografia ativa. O uso desses protocolos trarão o nível de segurança e robustez requeridos pelo projeto para acesso aos servidores.

O acesso ao sistema XPTO terá protocolo HTTPS com aquisição de certificado digital específico para essa finalidade. A criptografia existente nesse protocolo, garantirá a proteção dos dados trafegados.

Os acessos ao sistema por parte dos usuários é autenticado através do Entra ID corporativo com uso da autenticação de mútiplo fator. Os privilégios e papéis (roles) dos usuários dentro sistema são controlados por grupos criados dentro do Entra ID que serão lidos e interpretados pelo próprio sistema.

Adicionalmente os controladores de domínio do AD DS (Active Directory Domain Services) estão estrategicamente distribuídos entre a estrutura on premise e a infra em nuvem contemplando replicação entre si e sincronismo com o Entra ID.

O acesso às APIs do sistema hospedado em nuvem será controlado por API Gateway (Azure API Management). O uso do API gateway visa o reforço da segurança do sistema e também monitorar as solicitações, disponibilidade, respostas e eventuais erros das APIs.

##### 3.7. Monitoramento
O projeto contempla o uso de ferramenta para monitoramento das estruturas em nuvem e on premise.

Essencialmente a ferramenta de monitoramento tem como objetivos fazer a observabilidade da aplicação (volumes de acessos e transações, disponibilidade sistêmica, consumo de recursos, levantamento de indicadores, análise de comportamento, etc.) e analisar a disponibilidade da infraestrutura (servidores, bancos de dados, ativos de rede, etc.).

Por ser uma ferramenta open source com robustez considerável e recursos adequados às necessidades do projeto, o Prometheus fará o monitoramento dos ambientes coletando dados de telemetria para análise através do Grafana, o qual possui interface gráfica intuitiva que auxiliará na visualização dos dashboards e levantamento de indicadores requeridos para o negócio.

Considerando o tráfego de rede como um dos elementos diretamente relacionados ao desempenho do sistema, torna-se recomendável a implementação da observabilidade de rede que trará ganhos significativos para o cotidiano da XPTO, tais como: redução de indisponibilidade, otimização do desempenho de rede, maior controle da postura de segurança e pró atividade na resolução de problemas.

Para que os benefícios acima mencionados sejam obtidos, torna-se fundamental:
- Controlar as mudanças nos ambientes de rede;
- Manter a topologia de rede atualizada e alinhada com as melhores práticas de cyber segurança. Adicionalmente estabelecer um fluxo de comunicação e conscientização contínuos juntos aos profissionais responsáveis pela manutenção e monitoramento dos ambientes de rede;
- Medir constantemente a performance dos ambientes de rede com o objetivo de identificar eventuais gargalos e potenciais pontos de falha;
- Estabelecer um relacionamento entre dados (indicadores) de mudanças e performance da rede, o que permitirá aos profissionais envolvidos analisar causas raízes de problemas e incidentes ocorridos, bem como a forma de resolvê-los.
Para viabilizar uma observabilidade de rede eficaz e eficiente é importante que o Prometheus seja configurado para prover o rastreamento de mudanças na rede em tempo real e as condições de disponibilidade dos ambientes e ativos de rede com base na topologia vigente, o que fornecerá insumos suficientes para uma atuação pró ativa na análise e resolução de problemas e incidentes.

##### 3.8. Backup
Atendendo aos requisitos de continuidade do negócio e recuperação de desastres, foi definido o uso de uma solução para backup das máquinas virtuais e bancos de dados existentes no ambiente on premises e na estrutura em nuvem.

O MARS (Microsoft Azure Recovery Services) é um dos serviços disponíveis na Azure para backup sem a necessidade do provisionamento e gestão de infraestrutura dedicada. Ele executa o backup através de agentes a serem instalados nos servidores e armazena os dados copiados em vault dedicado com acesso restrito e criptografia.

##### 3.9. Infraestrutura de servidores (on premise)
O ambiente on premise contempla virtualização dos servidores do sistema com uso do VMware vSphere, um hypervisor robusto com funcionalidades de alta disponibilidade (H.A.) e otimização do uso de recursos computacionais (DRS), ou seja, o VMware é capaz de distribuir a carga de utilização dos recursos entre os hosts existentes, evitando eventuais sobrecargas e/ou ociosidade entre eles.

O armazenamento da estrutura é composto por dois storages configurados em cluster com arrays de discos com alta performance (SSD) configurados entre eles.

A infraestrutura de servidores do sistema hospedada na nuvem está disposta em máquinas virtuais, cluster kubernetes e computação serverless.

##### 3.10. Dimensionamento de Recursos
###### 3.10.1. Ambiente On Premise
###### 3.10.1.1. Hosts VMware
*Quantidade:* 2
*Modelo:* Dell PowerEdge R470
*CPU:* 1 x Intel Xeon 6 Efficient 6780E 2.2G com 144 núcleos
*Memória:* 512 GB

###### 3.10.1.2. Storage
*Quantidade:* 2
*Modelo:* Dell PowerStore 1200T
*Disco:* SSD NVMe 30 TB
*Configuração:* Cluster (H.A.)

###### 3.10.1.3. Máquinas Virtuais (Serviços Controle de Lançamento Diário e Consolidado)
*Quantidade:* 2
*vCPU:* 16
*Memória:* 30 GB

*Crescimento vertical (Host 1)*
*vCPU:* 20
*Memoria:* 48 GB

*Crescimento horizontal (estendido ao Host 2)*
*Quantidade:* 1
*vCPU:* 20
*Memoria:* 48 GB

###### 3.10.1.4. Máquinas Virtuais Domain Controllers
*Quantidade:* 2
*vCPU:* 2
*Memória:* 8 GB

*Crescimento vertical (Host 1)*
*vCPU:* 4
*Memoria:* 16 GB

*Crescimento horizontal (estendido ao Host 2)*
*Quantidade:* 1
*vCPU:* 4
*Memoria:* 16 GB

###### 3.10.1.5. Máquinas Virtuais Cluster SQL Server
*Quantidade:* 2
*vCPU:* 16
*Memória:* 32 GB

*Crescimento vertical (Host 1)*
*vCPU:* 24
*Memória:* 64 GB

*Crescimento horizontal (estendido ao Host 2)*
*Quantidade:* 1
*vCPU:* 24
*Memoria:* 64 GB

###### 3.10.2. Ambiente Cloud
###### 3.10.2.1. Managed Instance Azure SQL
*Quantidade:* 1
*vCPU:* 20 (Gen5)
*Memória:* 100 GB

*Crescimento vertical*
*vCPU:* 30 (Gen5)
*Memória:* 150 GB

###### 3.10.2.2. Máquinas Virtuais Domain Controllers
*Quantidade:* 2
*vCPU:* 2
*Memória:* 8 GB

*Crescimento vertical*
*vCPU:* 4
*Memória:* 16 GB

###### 3.10.2.3. Máquinas Virtuais Prometheus
*Quantidade:* 4
*vCPU:* 4
*Memória:* 16 GB

*Crescimento vertical*
*vCPU:* 8
*Memória:* 32 GB

###### 3.10.2.4. Máquina Virtual Grafana
*vCPU:* 4
*Memória:* 16 GB

*Crescimento vertical*
*vCPU:* 8
*Memória:* 32 GB

###### 3.10.2.5. Cluster Kubernetes
*Nodes*
*Quantidade:* 3
*vCPU:* 32
*Memória:* 64 GB

*Pods*
*Requests e Limits*
*CPU:* 0.2
*Memória:* 0.4 GB
*Quantidade média pods:* 250

###### 3.10.2.6. Functions
*Opção de hospedagem:* Plano Premium
*Inicialização a frio:* Não possui (Uma ou mais instâncias sempre ativas) 
*CPU (por instância):* 1- 4 vCPU
*Memória (instância):* 3,5 - 14 GB

###### 3.10.2.7. Azure Redis for Cache
*Tipo Cache:* Standard
*Memória:* 50 GB
*Quantidade de nodes:* 2

###### 3.10.3. Networking
###### 3.10.3.1. Links MPLS (Escritório)
*Quantidade:* 2
*Velocidade:* 500 Mbps

###### 3.10.3.2. Links MPLS (Nuvem x On Premise)
*Quantidade:* 2
*Velocidade:* 1 Gbps

###### 3.10.3.3. Links MPLS (Lojas)
*Quantidade:* 2
*Velocidade:* 500 Mbps

###### 3.10.3.4. Links Internet (Escritório)
*Quantidade:* 2
*Velocidade:* 500 Mbps

###### 3.10.3.5. Switch L3 (Ambiente On Premise)
*Modelo: Cisco Catalyst C9200-48PB*
*Quantidade:* 2
*Configuração:* H.A. (High Availability)
*Portas:* 48

###### 3.10.3.6. Firewalls (Ambiente Cloud)
*Modelo:* FortiGate Next-Generation Firewall for Azure (FortiGate-VM16 (FG-VM16))
*Quantidade:* 2
*Configuração:* H.A. (High Availability)
*vCPU:* 16

###### 3.10.3.7. Firewalls (Ambiente On Premise)
*Modelo:* FortiGate 200F
*Quantidade:* 2
*Configuração:* H.A. (High Availability)

###### 3.10.3.8. Firewalls (Escritório)
*Modelo:* FortiGate 101F
*Quantidade:* 4
*Configuração:* H.A. (High Availability)

###### 3.10.3.9. Firewalls (Lojas)
*Quantidade:* 1 por loja
*Modelo:* FortiGate 90G

##### 4. Estratégia de automação (IaC)
Em função da criticidade dos ambientes XPTO, recomenda-se o uso de scripts Terraform e playbooks Ansible no projeto para provisionamento e configuração automatizada dos recursos de infraestrutura.

Todos os scripts e playbooks serão armazenados em repositórios a serem configurados no Azure DevOps, plataforma escolhida pela facilidade de integração e uso conjunto com a nuvem Azure.

Principais benefícios ao projeto proporcionados pela automação no provisionamento e configuração de infraestrutura via IaC (Infraestrutura como Código):

***Agilidade e Flexibilidade:*** Substitui o provisionamento e configuração manual dos recursos de infraestrutura, os quais normalmente são mais demorados e sujeitos a erros. Além disso, permitem a criação, configuração e alteração de recursos sob demanda e em larga escala.

***Produtividade:*** Permite aumentar a produtividade do time, liberando-o de tarefas repetitivas e deixando-o com tempo livre para atuar em atividades mais estratégicas.

***Rastreabilidade:*** É possível implementar um controle das alterações de infraestrutura através do versionamento dos scripts e playbooks utilizados, o que aumenta a consistência do ambiente.

***Assertividade e Reutilização:*** Minimiza erros operacionais, uma vez que os scripts já estão prontos e validados para novas execuções a qualquer momento.

***Recuperação de desastres:*** Os scripts reduzem o tempo para recuperação de desastres em infraestrutura, pois sua execução é muito mais rápida do que o provisionamento manual.

##### 5. Estratégia FinOps
O uso da ferramenta IBM Turbonomic proporcionará a redução e gestão detalhada de custos com a estrutura em nuvem e o uso sustentável dos recursos no ambiente on premise mantendo altos níveis de desempenho do sistema sob demanda.

O IBM Turbonomic será utilizado na modalidade SaaS, não requerendo portanto o provisionamento de infraestrutura dedicada para a ferramenta.

Para que os benefícios acima mencionados sejam atingidos em conjunto com as práticas FinOps, recomenda-se a estratégia abaixo:

- Implementar um time FinOps com participantes dos times de TI e Finanças para divulgação e conscientização das práticas FinOps junto aos envolvidos (profissionais de TI, finanças, executivos e áreas de negócio);
- Definir orçamentos considerando sazonalidades do negócio;
- Determinar o SLO (Objetivo de Nível de Serviço) do sistema;
- Estabelecer uma política de tags para todos os recursos de infraestrutura;
- Implementar monitoramento contínuo através da ferramenta relacionado ao consumo de recursos;
- Criar alertas de consumo;
- Através do IBM Turbonomic analisar todas as dependências existentes entre os componentes de sistema e infraestrutura para identificação antecipada de gargalos;
- Analisar as recomendações do IBM Turbonomic para melhoria de desempenho do sistema e com base nelas definir ações automáticas na ferramenta para que ela possa provisionar e reduzir recursos com base na demanda do sistema em tempo real;
- Integrar o IBM Turbonomic ao Prometheus para coleta de métricas e dados de telemetria detalhados relacionados ao uso do sistema que auxiliarão a ferramenta na resolução de problemas com desempenho e na recomendação de melhorias.

##### 6. Plano de Disaster Recovery
###### 6.1. Objetivo
- Minimizar interrupções na operação da infraestrutura tecnológica da XPTO.
- Sustentar a operação do negócio.
- Estabelecer os procedimentos para uso do ambiente de contingência e seus mecanismos.
- Definir o processo de restauração do ambiente produtivo.
- Ser o guia de referência a todos os envolvidos na gestão técnica da infraestrutura de TI no tocante aos procedimentos e critérios para recuperação de desastres.

###### 6.2. Responsabilidades
***Estratégico:*** Diretoria
***Tático:*** Gestores de TI e Equipe de Governança
***Operacional:*** Equipes de Infraestrutura TI (Analistas) e Cyber Segurança (Analistas)

###### 6.3. Considerações
Sempre que o plano de recuperação de desastres for acionado, todos os responsáveis mencionados acima devem ser imediatamente comunicados.

O plano de recuperação de desastres deve ser testado e revalidado a cada 6 meses com participação das equipes de Governança, Infraestrutura de TI e Cyber Segurança.

Qualquer alteração a ser feita no plano de recuperação de desastres deve contar com a aprovação de todas as equipes envolvidas.

###### 6.3.1. RPO (Recovery Point Objective)
Definido em 2h00 considerando indisponibilidade simultânea dos ambientes em nuvem e on premise.
Para indisponibilidade em apenas um dos ambientes existentes foi definido um índice de 00h30.

###### 6.3.2. RTO (Recovery Time Objective) 
Definido em 24h00 considerando indisponibilidade simultânea dos ambientes em nuvem e on premise.
Para indisponibilidade em apenas um dos ambientes existentes foi definido um índice de 10h00.

###### 6.4. Sistemas
Sistema de fluxo de caixa XPTOSys tendo como serviços críticos ao negócio:
- Serviço de controle de lançamentos (back end)
- Serviço de consolidado diário (back end)
- Front end de acesso ao sistema através dos endereços: sys1.xpto.corp e sys2.xpto.corp

O sistema de fluxo de caixa e seus serviços críticos acima mencionados estão hospedados em dois ambientes distintos a saber: 
- Azure (site primário – produtivo) 
- Data Center XPTO (site secundário – produtivo e disaster recovery)

###### 6.5. Backup
As informações essenciais ao funcionamento do sistema são diariamente incluídas na rotina de backup que seguem as definições abaixo. Os dados envolvidos na rotina de backup compreendem bancos de dados, máquinas virtuais, configurações de firewalls e switches.

###### 6.5.1. Backup Diário
*Frequência:* de segunda a sábado
*Horários:* Executado a cada 2 horas para os bancos de dados e das 00h00 às 02h00 para os demais recursos (máquinas virtuais).
*Método da rotina de backup:* Incremental
*Mídia para backup dos dados:* Vault (Azure)
*Retenção:* 30 dias

###### 6.5.2. Backup Semanal
*Frequência:* domingo
*Horário:* das 00h00 às 06h00
*Método da rotina de backup:* Full
*Mídia para backup dos dados:* Vault (Azure)
*Retenção:* 1 ano

###### 6.5.3. Backup Mensal
*Frequência:* Primeiro dia útil do mês
*Horário:* das 00h00 às 06h00
*Método da rotina de backup:* Full
*Mídia para backup dos dados:* Vault (Azure)
*Retenção:* 5 anos

###### 6.6. Recuperação de desastres (Azure - Site primário Produtivo)
Para os recursos de infraestrutura (redes, máquinas virtuais, cluster kubernetes) não utilizados como PaaS (Platform as a Service), será necessário reconfigurá-los através dos scripts Terraform para que os mesmos sejam automaticamente provisionados. 

Caso os recursos afetados necessitem de configurações específicas (instalação e configuração de softwares, parâmetros de rede e configuração para sistemas operacionais) torna-se necessário executar os respectivos playbooks Ansible para que tais configurações sejam realizadas em modo automático.

Em seguida, o backup de dados dos recursos afetados deve ser restaurado.

Para os recursos utilizados como PaaS, será necessário solicitar ao cloud provider a restauração de dados a partir do backup existente e configurar os parâmetros requeridos.

Desastres irreversíveis ocorridos nos recursos de conectividade desse ambiente (firewalls e gateways de comunicação), requerem intervenção direta das equipes responsáveis para reinstalação e restauração de configurações dos firewalls a partir de backup existente e execução dos scripts Terraform para reconfiguração dos gateways de comunicação.

Desastres ocorridos nesse ambiente, exigem que os acessos ao sistema de fluxo de caixa seja feito exclusivamente através do ambiente on premise (Data Center XPTO).

###### 6.6.1. Restauração de acesso ao ambiente
Após a recuperação do ambiente, o sincronismo de dados existente com o ambiente on premise deve ser restabelecido. Ao término desse sincronismo, as equipes responsáveis devem testar e validar o ambiente para que os usuários possam acessá-lo.

###### 6.7. Recuperação de desastres (Data Center XPTO – Site secundário Produtivo e Disaster Recovery)
###### 6.7.1. Falhas de software
Será necessário reconfigurar os recursos de infraestrutura afetados através dos scripts Terraform para que os mesmos sejam automaticamente provisionados.

Caso os recursos afetados necessitem de configurações específicas (instalação e configuração de softwares, parâmetros de rede e configuração para sistemas operacionais) torna-se necessário executar os respectivos playbooks Ansible para que tais configurações sejam realizadas em modo automático.

Em seguida, o backup de dados dos recursos afetados deve ser restaurado.
Desastres ocorridos nesse ambiente, exigem que os acessos ao sistema de fluxo de caixa seja feito exclusivamente através do ambiente em nuvem Azure.

###### 6.7.2. Falhas de hardware
Desastres irreversíveis ocorridos nos recursos de conectividade desse ambiente (firewalls e switches), requerem intervenção direta das equipes responsáveis para reinstalação e restauração de configurações dos mesmos a partir de backup existente.

Em caso de falhas de hardware generalizadas em servidores e storages, as equipes responsáveis deverão atuar diretamente no reparo, reinstalação e restauração de configurações dos mesmos a partir de backup existente.

###### 6.7.3. Restauração de acesso ao ambiente
Após a recuperação do ambiente, o sincronismo de dados existente com a estrutura em nuvem deve ser restabelecido. Ao término desse sincronismo, as equipes responsáveis devem testar e validar o ambiente para que os usuários possam acessá-lo.

###### 6.8. Conectividade do escritório matriz e lojas
Em caso de indisponibilidade em um dos circuitos Express Route, o escritório deverá utilizar o circuito que permanecer ativo para acesso aos ambientes em nuvem e on premise.

Em caso de indisponibilidade em um dos circuitos Express Route, as lojas devem utilizar o circuito que permanecer ativo para acesso ao sistema.

O retorno da comunicação do circuito Express Route que apresentar falha não requer nenhuma intervenção técnica nos acessos porque eles são configurados em modo failover. 

###### 6.9. Conectividade entre o ambiente on premise e estrutura em nuvem
Em caso de indisponibilidade em um dos circuitos Express Route, a comunicação entre esses ambientes continuará ocorrendo através do circuito que permanecer ativo.

O retorno da comunicação do circuito Express Route que apresentar falha não requer nenhuma intervenção técnica porque eles são configurados em modo failover.

##### 7. Aplicação do Modelo OSI no diagnóstico e resolução dos problemas de rede do projeto
Por tratar-se de um padrão para comunicação entre diferentes sistemas computacionais, o Modelo OSI pode ser utilizado para análise e solução de problemas na infraestrutura de rede do projeto. Abaixo é descrito o relacionamento das camadas do Modelo OSI com os componentes e protocolos utilizados no projeto:

###### Camada 1 – Física
Indicada para resolução de problemas nos ativos de rede (switches, roteadores, interfaces de rede dos servidores, firewalls, balanceadores de carga e gateways de comunicação), cabos de interconexão e links de comunicação.

###### Camada 2 - Enlace de dados
Utilizada para análise e teste da comunicação entre os dispositivos dentro da mesma rede. Um exemplo prático referente ao projeto seria um teste de comunicação entre servidores dentro da rede do ambiente on premise.

###### Camada 3 – Rede
Responsável pela análise e teste da comunicação entre redes diferentes. Um exemplo prático referente ao projeto seria um teste de comunicação entre a rede do ambiente on premise e a rede da estrutura em nuvem.

###### Camada 4 – Transporte
Utilizada para análise de comunicação entre dispositivos e pelo controle de fluxo e erros. Um exemplo prático aplicado ao projeto seria analisar a latência de comunicação entre 2 servidores através do comando “tracert”.

###### Camada 5 – Sessão
Responsável pela abertura e fechamento de comunicação entre dois dispositivos e por sincronizar e verificar transferência de dados. Um exemplo prático aplicado ao projeto seria a análise da duração e dados transferidos em uma sessão entre o serviço de controle de lançamento e a estação de trabalho de um determinado usuário devido a um erro reportado por ele ao efetuar um lançamento no sistema.

###### Camada 6 – Apresentação
Utilizada para preparação dos dados que serão utilizados pela camada de aplicação. Um exemplo prático aplicado ao projeto seria a verificação de erros no acesso de um Analista de Infra a um determinado servidor via SSH devido a erros de configuração no par de chaves usado entre a estação de trabalho do analista e o servidor. 

###### Camada 7 – Aplicação
Responsável pelo tipo da aplicação e seus métodos de comunicação. Um exemplo prático aplicado ao projeto seria o navegador do usuário comunicnado-se com os servidores do sistema utilizando o protocolo https. 
