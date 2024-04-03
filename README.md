# Documento Técnico: Market Stalker

## 1. Introdução

Market Stalker visa revolucionar o modo como consumidores planejam e executam suas compras em supermercados, fornecendo uma plataforma que não só permite a comparação de preços em tempo real, mas também otimiza a jornada de compras com base na localização e preferências do usuário. Este documento delineia as funcionalidades chave oferecidas aos usuários e aos supermercados, bem como os fluxos detalhados de interação dentro do aplicativo.

## 2. Arquitetura Geral do Sistema

A arquitetura do sistema é baseada em microserviços, utilizando Docker para containerização e AWS Lightsail para hospedagem.


```plaintext
                    +----------------+      +----------------+        
                    |                |      |                |
                    |    Usuário     |      |    Mercado     |
                    |                |      |                |
                    +-------|--------+      +--------|-------+           
                            | (1)                    | (7)
                            V                        V
                    +-------|--------+      +--------|-------+
                    | App Mobile(UI) |      |   App Web(UI)  |
                    |  React Native  |      |     NextJS     |
                    +-------|--------+      +--------|-------+
                            | (2)                    | (8)
                            ⇅                        ⇅ 
                    +-------|------------------------|-------+       
                    |               API Gateway              |  
                    +-------------------|--------------------+ 
                                        | (3)                    
                                        ⇅                         
        +-------------------------------|-------------------------------+
        |                   ____________|                               |
        |                   |                                           |
        |                   |    AWS Lightsail                          |
        |               (4) |    (Ubuntu 2204)                          |
        |                   |                                           |
        |   +---------------|--------+     +------------------------+   |
        |   |                        | (5) |                        |   |
        |   |    Docker Container    -->--->    Docker Container    |   |
        |   |                        <---<--                        |   |
        |   |    Backend Node.Js     | (6) | PostgresSQL (Supabase) |   |
        |   |                        |     |                        |   |
        |   +------------------------+     +------------------------+   |
        |                                                               |
        |                                                               |
        +---------------------------------------------------------------+ 

```

#### Legenda do Fluxograma

- **(1) Usuário interage com o App Móvel:** O usuário acessa funcionalidades como pesquisa de preços, criação de listas de compras, e comparação de produtos através do aplicativo móvel desenvolvido em React Native.

- **(2) Comunicação do App Móvel com a API Gateway:** O aplicativo móvel envia solicitações à API Gateway para realizar operações como buscar preços atualizados, obter detalhes de produtos, ou atualizar a lista de compras.

- **(7) Mercado interage com o App Web:** Representantes dos mercados acessam o dashboard web para gerenciar informações sobre seus produtos, visualizar análises de vendas, e ajustar estratégias de precificação.

- **(8) Comunicação do App Web com a API Gateway:** O dashboard web envia solicitações à API Gateway para realizar operações como atualizar preços de produtos, modificar informações de itens, ou acessar relatórios de tendências de compras.

- **(3) API Gateway encaminha solicitações:** Funciona como intermediário que recebe solicitações tanto do aplicativo móvel quanto do dashboard web, encaminhando-as aos serviços correspondentes no backend.

- **(4) Serviços na AWS Lightsail recebem as solicitações:** A infraestrutura na AWS Lightsail, contendo serviços de backend e banco de dados em containers Docker separados, recebe as solicitações.

- **(5) Backend Node.js processa as solicitações:** O backend Node.js, hospedado em um container Docker, processa as solicitações, realizando operações de dados, lógica de negócios, e interagindo com o banco de dados.

- **(6) Interação com o PostgreSQL (Supabase):** Para operações de banco de dados, o backend se comunica com o PostgreSQL gerenciado pelo Supabase, que retorna os dados solicitados, encaminhando as respostas de volta à origem da solicitação através da API Gateway.

### 2.1 Aplicativo Móvel (Consumidores)

- **Tecnologia:** React Native
- **Funcionalidades:** Listagem de produtos, comparação de preços, seleção de supermercados, preferências de compra.

### 2.2 Backend (API e Serviços)

- **Hospedagem:** AWS Lightsail com Docker
- **Tecnologias:**
  - **Node.js:** Para lógica de negócios e integração de APIs.
  - **Supabase:** Autenticação, armazenamento, funções serverless, atualizações em tempo real, e PostgREST para APIs RESTful.

### 2.3 Dashboard (Supermercados)

- **Tecnologia:** Next.js
- **Funcionalidades:** Visualização de dados de consumo, gestão de ofertas, insights de mercado.
- **Integração:** Acesso direto ao backend para consulta e gestão de dados.

## 3. Estratégia de Dados

### Coleta de Dados

Utiliza-se técnicas de web scraping e integração com APIs de supermercados para a coleta de dados, que são processados, armazenados e disponibilizados através do Supabase.

### Atualização e Armazenamento

O serviço Node.js no backend é responsável pela atualização periódica dos dados no banco PostgreSQL, utilizando técnicas como cron jobs para scraping programado.

## 4. Estratégia de Monetização

O modelo de monetização é baseado em assinaturas para supermercados, oferecendo destaque de produtos e acesso a análises detalhadas de comportamento de compra dos usuários.

## 5. Infraestrutura e Segurança

### Containers Docker

Cada componente do sistema (aplicativo, backend, dashboard) é encapsulado em seu próprio container Docker, facilitando a manutenção, escalabilidade e segurança.

### Conformidade e Segurança

Adota-se HTTPS para todas as comunicações externas. O acesso aos dados é controlado rigorosamente através de políticas de segurança do Supabase e autenticação JWT.

## 6. Funcionalidades do Usuário

### 6.1 Criação e Gerenciamento de Listas de Compras Personalizadas
Usuários podem criar listas de compras nomeadas e adicionar produtos a estas listas. Cada lista pode ser associada a diferentes supermercados, permitindo comparações de preços específicas para cada combinação de lista e supermercado.

### 6.2 Comparação de Preços e Seleção de Supermercados
Ao visualizar uma lista de compras, o usuário pode selecionar supermercados próximos para comparar os preços dos itens listados. Isso facilita a decisão sobre qual supermercado visitar para maximizar economia.

### 6.3 Organização por Grupos e Rota Personalizada
Produtos dentro das listas podem ser agrupados por categorias, como "Laticínios" ou "Frutas". Com base nisso e na localização dos supermercados escolhidos, o aplicativo sugere uma rota otimizada de compras, integrando-se a serviços de mapas externos para direcionamento.

### 6.4 Notificações de Localização e Progresso de Compras
O aplicativo detecta a chegada do usuário a um supermercado e monitora o tempo estimado de permanência, baseado na quantidade de itens a serem comprados. Notificações são enviadas para lembrar o usuário de marcar grupos de itens como comprados. Conseguindo medir o tempo em cada lugar mesmo que o usuário não marque nada (com base na localização ainda).

### 6.5 Feed de Ofertas Relâmpago
Usuários e mercados podem postar ofertas relâmpago no feed do aplicativo. Usuários fazem isso escaneando o código de barras de um produto em oferta e adicionando o preço promocional. As postagens no feed são reguladas para assegurar a qualidade das informações.

## 7. Funcionalidades do Mercado

### 7.1 Dashboard de Gerenciamento e Análise
Supermercados acessam um dashboard avançado para gerenciar suas informações de produto, visualizar dados analíticos de consumo, e criar estratégias de precificação baseadas em análises detalhadas dos hábitos de compra dos usuários.

### 7.2 Relatórios Avançados e Insights de Mercado
Os mercados têm acesso a relatórios que cruzam diversas fontes de dados, oferecendo insights sobre preferências de compra, eficácia de ofertas relâmpago, e tendências de consumo.

### 7.3 Destaque de Produtos e Ofertas
Como funcionalidade premium, os mercados podem optar por destacar produtos específicos dentro do aplicativo, aumentando sua visibilidade entre os usuários.

### 7.4 Criação de Listas Pré-Definidas e Ofertas Diretas
Mercados parceiros podem criar listas de compras sugeridas baseada nos produtos que quiser, mas principalmente os que estão em promoção, incentivando os usuários a visitar suas lojas para aproveitar preços vantajosos.

## 8. Localização e Roteirização Inteligente

O utiliza a localização atual do usuário para sugerir supermercados próximos e organizar a lista de opções por proximidade. A roteirização considera não só a distância, mas também a otimização baseada no número de itens a serem comprados em cada local.

## 9. Roadmap de Desenvolvimento

O desenvolvimento futuro do Market Stalker inclui a implementação de análises em tempo real da lotação dos supermercados com base nos usuários ativos dentro da localização de um mercado específico. Expansão das funcionalidades do APP, permitindo fazer a leitura de qualquer código de barras e já localizando o produto e seus preços no mercados já definidos pelo usuario, além de uma melhor definição da gestão de ofertas em destaque para mercados parceiros.

## 10. Conclusão

O "Market Stalker" visa transformar a experiência de compra, fornecendo aos usuários uma ferramenta poderosa para economizar em suas compras, enquanto oferece aos supermercados insights valiosos sobre as preferências dos consumidores.
