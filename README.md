# Projeto de Escalabilidade Automática e Balanceamento de Carga

Este repositório documenta um projeto para explorar e implementar conceitos de Auto Scaling, Load Balancer e regras de escalonamento (Scaling Up e Scaling Down) em um ambiente de nuvem.

## Resumo

O objetivo deste projeto é demonstrar como o Auto Scaling e o Load Balancing trabalham em conjunto para criar uma arquitetura de aplicação robusta, resiliente e com custos otimizados. Através da configuração desses serviços, uma aplicação pode se adaptar dinamicamente às flutuações de tráfego, garantindo alta disponibilidade e performance, ao mesmo tempo em que minimiza o desperdício de recursos.

---

## 1. O que é cada componente?

### Auto Scaling

O Auto Scaling (ou escalonamento automático) é um serviço que ajusta automaticamente a quantidade de recursos computacionais (como servidores virtuais) em uma aplicação, de acordo com a demanda em tempo real. O objetivo é manter a performance da aplicação de forma consistente e previsível, com o menor custo possível.

- **Como funciona?** Ele monitora métricas da aplicação, como a utilização da CPU, o número de requisições por minuto, ou a latência de rede. Com base em regras pré-definidas, ele adiciona mais recursos quando a demanda aumenta (Scaling Up/Out) e remove recursos quando a demanda diminui (Scaling Down/In).

### Load Balancer

O Load Balancer (ou balanceador de carga) atua como um "controlador de tráfego" para os servidores da sua aplicação. Ele distribui as requisições de entrada entre múltiplos servidores, evitando que um único servidor fique sobrecarregado.

- **Como funciona?** Ao receber uma requisição, o Load Balancer a encaminha para um dos servidores disponíveis em um "grupo de destino" (Target Group), com base em um algoritmo de balanceamento (por exemplo, distribuindo igualmente as requisições). Ele também realiza verificações de saúde (Health Checks) para garantir que as requisições sejam enviadas apenas para servidores que estão operando corretamente.

### Regras de Scaling Up e Down

As regras de Scaling Up (escalar para cima/fora) e Scaling Down (escalar para baixo/dentro) são as políticas que o serviço de Auto Scaling utiliza para decidir quando adicionar ou remover servidores.

- **Scaling Up (Scale Out):** É o processo de adicionar mais instâncias de servidores em resposta a um aumento na demanda. Uma regra comum de Scaling Up é acionada quando a utilização média da CPU de todos os servidores ultrapassa um determinado limite (por exemplo, 70%) por um período de tempo específico.

- **Scaling Down (Scale In):** É o processo de remover instâncias de servidores quando a demanda diminui, para economizar custos. Uma regra típica de Scaling Down pode ser acionada quando a utilização média da CPU cai abaixo de um certo patamar (por exemplo, 30%).

---

## 2. O que aprendemos com este projeto?

A implementação prática de Auto Scaling e Load Balancing proporcionou valiosos aprendizados sobre a criação de arquiteturas de nuvem modernas e eficientes:

- **Resiliência e Alta Disponibilidade:** A combinação desses dois serviços é fundamental para a tolerância a falhas. Se um servidor se torna indisponível, o Load Balancer para de enviar tráfego para ele, e o Auto Scaling pode substituí-lo automaticamente por uma nova instância saudável, garantindo que a aplicação continue online.

- **Otimização de Custos:** O principal benefício do Auto Scaling é a economia. Em vez de manter uma grande quantidade de servidores ociosos para picos de tráfego, a infraestrutura se ajusta dinamicamente, garantindo que você pague apenas pelos recursos que realmente utiliza.

- **Performance e Experiência do Usuário:** Em momentos de alta demanda, o Auto Scaling garante que haja recursos suficientes para atender a todos os usuários sem degradação da performance. O Load Balancer, por sua vez, distribui a carga de forma eficiente, evitando gargalos e lentidão.

- **Automação e Eficiência Operacional:** Uma vez configuradas, as regras de escalonamento e o balanceamento de carga operam de forma autônoma, reduzindo a necessidade de intervenção manual da equipe de operações para gerenciar picos de tráfego ou falhas de instâncias.

- **Importância das Métricas:** A escolha das métricas corretas para acionar as regras de escalonamento é crucial. Uma configuração inadequada pode levar a um escalonamento muito lento (prejudicando a performance) ou muito agressivo (aumentando os custos desnecessariamente).

---

## 3. Passo a passo de cada serviço

A seguir, apresentamos um guia conceitual e generalizado dos passos para configurar cada um dos serviços. A terminologia exata pode variar entre provedores de nuvem (como AWS, Azure ou Google Cloud).

### Configurando um Load Balancer

1.  **Criar um Grupo de Destino (Target Group):**
    * Defina um grupo que conterá os servidores que receberão o tráfego.
    * Especifique o protocolo (HTTP/HTTPS) e a porta que sua aplicação utiliza (ex: porta 80).
    * Configure as **verificações de saúde (Health Checks)**. O Load Balancer usará esse endpoint (ex: `/health`) para verificar se os servidores estão operacionais.

2.  **Criar o Load Balancer:**
    * Escolha o tipo de Load Balancer (por exemplo, Application Load Balancer para tráfego web).
    * Selecione as redes e sub-redes onde ele atuará. É crucial que ele opere em múltiplas zonas de disponibilidade para garantir a resiliência.
    * Configure os **"Ouvintes" (Listeners)**. Um Listener verifica as requisições de entrada em uma porta específica (ex: porta 80 para HTTP) e as encaminha para o seu Grupo de Destino.

### Configurando o Auto Scaling

1.  **Criar um Modelo de Lançamento (Launch Template/Configuration):**
    * Defina um modelo para os servidores que serão criados pelo Auto Scaling.
    * Especifique a imagem do sistema operacional (AMI), o tipo de instância (ex: `t2.micro`), as configurações de rede e segurança (Security Groups) e, se necessário, um script de inicialização para instalar sua aplicação.

2.  **Criar o Grupo de Auto Scaling (Auto Scaling Group):**
    * Associe o Modelo de Lançamento criado no passo anterior.
    * Defina a capacidade desejada, mínima e máxima de instâncias. Por exemplo:
        * **Mínimo:** 1 (para garantir que sempre haja pelo menos um servidor rodando).
        * **Desejado:** 2 (a quantidade inicial de servidores).
        * **Máximo:** 10 (o limite para evitar custos excessivos).
    * Associe o Grupo de Auto Scaling ao Grupo de Destino do seu Load Balancer. Isso garante que toda nova instância seja automaticamente registrada no Load Balancer para receber tráfego.

3.  **Definir as Políticas de Escalonamento (Scaling Policies):**
    * **Política de Scaling Up (Scale Out):**
        * **Tipo:** Escalonamento por passos ou por alvo.
        * **Métrica:** Utilização média de CPU.
        * **Alvo/Limite:** 70%.
        * **Ação:** Adicionar 1 nova instância.
    * **Política de Scaling Down (Scale In):**
        * **Métrica:** Utilização média de CPU.
        * **Alvo/Limite:** 30%.
        * **Ação:** Remover 1 instância.
    * É recomendável configurar um **período de "cooldown"** para evitar que o Auto Scaling adicione ou remova instâncias de forma muito agressiva em resposta a flutuações temporárias de tráfego.

Com esses componentes configurados, a sua aplicação estará pronta para escalar de forma automática e resiliente, gerenciando o tráfego de forma eficiente e otimizando os custos operacionais.
