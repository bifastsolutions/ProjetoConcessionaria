# Desafio

Uma concessionária de veículos enfrentava dificuldades para controlar as vendas e devoluções de seus carros, o que resultava em prejuízos.
Os gestores não tinham acesso às informações em tempo hábil para tomar decisões estratégicas sobre quais carros comprar e quais modelos estavam tendo as maiores taxas de devolução.
 
Os gestores reconheceram que precisavam de uma solução de análise de dados e inteligência de negócios para obter visibilidade de em tempo hábil
do desempenho de cada modelo, as preferências dos clientes e as taxas de devolução. Eles estavam buscando um parceiro para implementar a solução
e ajudá-los a tomar decisões mais assertivas sobre faturamento, unidades vendidas, ticket médio, taxa de devolução entre outros KPIs importante para os gestores.
 
Eles já possuíam um sistema online para visualização dos dados das vendas e devoluções da concessionária de forma básica, porém, o processo de extrair,
tratar e visualizar esses dados de forma inteligente era um grande problema para a empresa.

# Quais a principais perguntas que devem ser respondidas (qual é a dor?)

## Indicadores

- Qual é o faturamento mensal da concessionária de veículos?
- Qual é a taxa de devolução por modelo de veículo?
- Qual é o ticket médio das vendas de veículos no último mês?
- Quantas unidades de veículos foram vendidas no último ano?
- Qual é o faturamento da concessionária em relação à meta estabelecida para o mês?
- Qual é o modelo de veículo com o maior número de vendas no último ano?

## Benefícios(valor) esperado pelo cliente

- Serei capaz de tomar decisões mais rápidas e em tempo hábil?
  R: Sim, ao ter acesso a dados precisos e atualizados, você será capaz de tomar decisões mais rápidas e eficazes, permitindo uma resposta ágil às necessidades do mercado.
- Essa solução irá automatizar o processo evitando a intervenção humano para coletar, tratar e visualisar esses dados?
  R: Sim, essa solução foi projetada para automatizar o processo, eliminando a necessidade de intervenção humana na coleta, tratamento e visualização dos dados, garantindo eficiência e precisão.
- Os meus dados ficaram seguros e disponíveis com qualidade?
  R: Absolutamente, os dados serão protegidos, mantidos com qualidade e disponíveis para análise, proporcionando segurança e confiabilidade na tomada de decisões.
- Irei economizar em relação a custos com infraestrutura?
  R: Sim, ao automatizar o processo, você poderá economizar significativamente em custos de infraestrutura, reduzindo a dependência de recursos humanos e aumentando a eficiência operacional.
- Irei conseguir alcançar meus objetivos comerciais?
  R: Definitivamente, a solução visa aprimorar suas estratégias comerciais, oferecendo insights valiosos baseados em dados confiáveis para alcançar e superar seus objetivos de negócio.
- Essa solução irá me ajudar a melhorar a experiência do meu cliente?
  R: Sim, ao melhorar a precisão das decisões e estratégias, a solução contribuirá para aprimorar a experiência do cliente, atendendo melhor às necessidades do mercado e oferecendo um serviço mais alinhado e eficiente.


# Resumo Arquitetura e pipeline de dados

![Arquitetura Projeto](https://github.com/bifastsolutions/ProjetoConcessionaria/assets/134235178/c18a5d97-05e4-4843-b652-edae330dcc02)

# Desenvolvimento da solução

Para armazenar os dados, escolhi o AWS S3, que é um serviço de armazenamento na nuvem altamente escalável e seguro.
Nele foi desenvolvido um data lake, com pastas dentro do Bucket dividindo em 3 zonas: Raw data(bronze zone), tratamentos gerais (Silver zone)
e refinamento e dados de qualidade (Gold zone). Cada pasta representa uma tabela extraída da API separadas de forma organizada em arquivos .csv.

<img src="https://user-images.githubusercontent.com/66785163/235541051-bc5b2e71-983e-48c5-bfd3-888dedc3822e.png" alt="DataLake" width="1000"/>

<p align="center"><b>Exemplo da camada Bronze dentro da AWS</b></p>



Utilizei Python para escrever scripts do processo de ETL/ELT (Extract, Transform and Load) que me permitiram extrair os dados do sistema via API,
transformá-los e tratá-los de acordo com as necessidades do negócio, e, em seguida, carregá-los em um repositório de dados (AWS S3).

Para armazenamento dos scripts do processo de extração, tratamento e carga dos dados citados acima, utilizei o AWS Glue, um serviço de ETL gerenciado que executa automaticamente tarefas de tratamento de dados em larga escala. Com isso, consegui limpar, enriquecer e transformar os dados de forma eficiente, garantindo que estivessem prontos para uso posterior. Dentro do Glue os scripts python para extração de tratamento das tabelas foram armazenados e feitos separadamente e os agendamento das rotinas para coleta de dados novos realizados pelo próprio scheduler nativo do Glue, usando Cron job para definição dos horários e quantidade de atualizações por dia.

## Exemplo do código usado para extrair os dados da API do sistema e armazenar no S3

```python
import requests
import pandas as pd
import boto3

# Configurações da AWS
BUCKET_NAME = 'lake-cars-project-prod'
AWS_ACCESS_KEY_ID = 'seu_access_key_id'
AWS_SECRET_ACCESS_KEY = 'seu_secret_access_key'

# Configurações da API
URL = 'enpoint da sua API'
HEADERS = {'Content-Type': 'application/json', 'Authorization': 'Bearer seu_token'}

# Conecta na API e normaliza os dados JSON
response = requests.get(URL, headers=HEADERS)
data = response.json()
df = pd.json_normalize(data)

# Conecta na AWS S3
s3 = boto3.client('s3', aws_access_key_id=AWS_ACCESS_KEY_ID, aws_secret_access_key=AWS_SECRET_ACCESS_KEY)

# Salva os dados em um arquivo CSV
csv_buffer = df.to_csv(index=False, encoding='utf-8', sep=';')
s3.Bucket('lake-cars-project-prod').upload_file(Filename=('raw_vendas.csv'),Key='bronze-zone/raw_data_vendas/' + 'raw_data_vendas.csv')
```
1. As bibliotecas requests, pandas e boto3 são importadas.
2. São definidos o nome do bucket da AWS, a chave de acesso e a chave secreta.
3. É definida a URL da API e as credenciais de autenticação, neste caso, um token de acesso(cada API é única, aqui é um exemplo, verifique a documentação da sua).
4. Uma requisição é feita para a URL da API usando o método get do objeto requests. A resposta é armazenada em response.
5. O conteúdo JSON da resposta é convertido em um objeto pandas DataFrame usando pd.json_normalize().
6. Uma conexão é estabelecida com o serviço S3 da AWS usando o objeto boto3.client(). As credenciais de acesso são passadas como argumentos.
7. O DataFrame é convertido em um objeto csv_buffer usando o método to_csv() e, em seguida, o arquivo CSV é carregado no bucket do S3 usando o método upload_file() do objeto S3.

<span style="color:red;">**Não esqueça de substituir também o caminho e o nome do arquivo**</span>

## Exemplo do código usado para tratamento de dados na camada Silver e armazenar no S3

```python
import pandas as pd
import boto3

# Configurações da AWS
BUCKET_NAME = 'lake-cars-project-prod'
AWS_ACCESS_KEY_ID = 'seu_access_key_id'
AWS_SECRET_ACCESS_KEY = 'seu_secret_access_key'

# Carrega os dados do arquivo CSV
df = pd.read_csv('bronze-zone/raw_data_vendas/raw_vendas.csv', usecols=['IDProduto', 'IDloja', 'Status', 'Qtde', 'Valor', 'Data', 'IDCliente'])

# Transforma as colunas conforme especificação
df['IDProduto'] = df['IDProduto'].astype(str)
df['Status'] = df['Status'].str.capitalize()
df['Valor'] = df['Valor'].str.replace(',', '.').str.strip()
df['date'] = pd.to_datetime(df['date']).dt.strftime('%d-%m-%Y')
df['IDCliente'] = df['IDCliente'].astype(str)

# Conecta na AWS S3 e salva o arquivo tratado
s3 = boto3.resource('s3', aws_access_key_id=AWS_ACCESS_KEY_ID, aws_secret_access_key=AWS_SECRET_ACCESS_KEY)
csv_buffer = df.to_csv(index=False, encoding='utf-8', sep=';')
s3.Bucket(lake-cars-project-prod).upload_file(Filename='silver_vendas.csv', Key='silver-zone/silver_vendas/silver_vendas.csv')
```
1. As bibliotecas pandas e boto3 são importadas.
2. São definidas as configurações da AWS, incluindo o nome do bucket, a chave de acesso e a chave secreta. Observe que os valores são fictícios(chave de acesso e chave secreta).
3. Os dados são carregados do arquivo CSV, que está localizado no caminho bronze-zone/raw_data_vendas/raw_vendas.csv. São selecionadas apenas as colunas IDProduto, IDloja, Status, Qtde, Valor, Data e IDCliente.
4. A coluna IDProduto é convertida para string, A primeira letra da coluna Status é transformada em maiúscula, A vírgula da coluna Valor é substituída por um ponto e quaisquer espaços em branco são removidos, A coluna Data é convertida para o formato de data dd-mm-aaaa, A coluna IDCliente é convertida para string.
5. A conexão é estabelecida com o serviço S3 da AWS, usando as credenciais fornecidas.
6. O DataFrame é convertido em um objeto csv_buffer usando o método to_csv() e, em seguida, o arquivo CSV é carregado no bucket do S3 usando o método upload_file() do objeto S3.


## Exemplo do código usado para a camada gold e armazenar no S3

```python
import pandas as pd
import boto3

# Configurações da AWS
BUCKET_NAME = 'lake-cars-project-prod'
AWS_ACCESS_KEY_ID = 'seu_access_key_id'
AWS_SECRET_ACCESS_KEY = 'seu_secret_access_key'

# Carrega os dados do arquivo CSV de vendas e devoluções
vendas = pd.read_csv('silver-zone/silver_vendas/silver_vendas.csv', usecols=['IDProduto', 'IDloja', 'Status', 'Qtde', 'Valor', 'date', 'IDCliente'])
devolucoes = pd.read_csv('silver-zone/silver_devolucoes/silver_devolucoes.csv', usecols=['IDCliente', 'Status'])

# Faz a junção das tabelas de vendas e devoluções
df = pd.merge(vendas, devolucoes, on='IDCliente', how='left')

# Conecta na AWS S3 e salva o arquivo tratado
s3 = boto3.client('s3', aws_access_key_id=AWS_ACCESS_KEY_ID, aws_secret_access_key=AWS_SECRET_ACCESS_KEY)
csv_buffer = df.to_csv(index=False, encoding='utf-8', sep=';')
s3.Bucket(lake-cars-project-prod).upload_file(Filename='fVendas.csv', Key='gold-zone/fVendas/fVendas.csv')
```
1. Importação de bibliotecas: As bibliotecas pandas e boto3 são importadas.
2. São definidas as configurações da AWS, incluindo o nome do bucket, a chave de acesso e a chave secreta. Observe que os valores são fictícios(chave de acesso e chave secreta).
3. O próximo passo é carregar os dados das vendas e devoluções em dois dataframes diferentes, usando a função read_csv() do pandas. Apenas as colunas necessárias são carregadas, especificadas pelo parâmetro usecols.
4. Os dois dataframes são unidos usando a função merge() do pandas, com a coluna IDCliente como chave de junção. A operação é um left join, o que significa que todas as vendas serão mantidas, mesmo que não haja uma devolução correspondente.
5. O resultado da união é atribuído a um novo dataframe df.
6. O DataFrame é convertido em um objeto csv_buffer usando o método to_csv() e, em seguida, o arquivo CSV é carregado no bucket do S3 usando o método upload_file() do objeto S3.

## Jobs no Glue(bronze-zone)

<img src="https://user-images.githubusercontent.com/66785163/235887900-e470d87c-d039-48d8-ad97-f4817520e679.png" alt="Jobs" width="1000"/>


## Pontos importantes para a utilização do  AWS Glue

É necessário criar uma policy no IAM e atrelar essa policy a uma role, essa role precisa ter um nome específico para funcionar no Glue não pode ser qualquer nome, nesse caso deve-se usar o nome **_AWSGlueServiceRoleDefault_** , no meu caso a configuração da policy ficou da seguinte forma:

<img src="https://user-images.githubusercontent.com/66785163/235888851-b6b1025e-bb36-4605-86ec-5f64823ab109.png" alt="Policy" width="1000"/>

A automatização para que os scripts dos jobs rodem em períodos determinados para enviar novos dados para o datalake é feito através do schedule nativo do Glue, onde no meu caso eu defini o range através de Cron Job com o tempo customizável, outro detalhe importante é que o horário é baseado no UTC. Por exemplo, se você estiver no Brasil e quiser que o job seja executado diariamente às 9:00 da manhã, no horário de Brasília, você precisa agendar o job para as 12:00 UTC, porque Brasília está 3 horas à frente do horário UTC.

<img src="https://user-images.githubusercontent.com/66785163/235891297-4d49b338-aa00-4bec-9b14-309633f58058.png" alt="CronJob" width="1000"/>

# AWS Athena

Para armazenar e consultar os dados tratados, usei o AWS Athena, que é um serviço de consulta em nuvem que me permitiu executar consultas SQL complexas. Embora o Athena não seja tecnicamente um data warehouse, ele é frequentemente usado como tal por empresas que desejam armazenar e analisar grandes quantidades de dados sem gastar muito dinheiro em infraestrutura. O Athena é altamente escalável e não requer nenhum servidor para ser configurado e gerenciado, tornando-o uma opção atraente para muitas empresas. Além disso, a conexão entre o S3 e o Athena é feita através de um crawler que cria tabelas com base nos arquivos do bucket S3, o que também cria um catálogo de dados. Isso torna o processo de consulta ainda mais fácil e rápido, permitindo que os usuários acessem os dados de maneira mais eficiente e com maior rapidez.

<img src="https://user-images.githubusercontent.com/66785163/236069050-71637b97-2bfb-4752-94aa-066ee8ed3d87.png" alt="Athena" width="1000"/>


Para visualizar os dados de maneira intuitiva e clara, usei o Power BI, uma ferramenta de visualização de dados altamente interativa e personalizável. Para garantir que as informações no Power BI Online estejam sempre atualizadas e prontas para uso, foi necessário agendar e executar as atualizações de dados através do Gateway de Dados do Power BI. Embora a atualização dos dados do Data Lake possa ser feita através do serviço AWS Glue, a atualização dos dados no Power BI Online precisa ser feita através do Gateway de Dados do Power BI.

# Power BI

Foi utilizado o modelo Star Schema, desde a construção e tratamento das tabelas na AWS dentro do data warehouse já foi pensando nos relacionamentos e modelo dentro do Power BI. Além disso, foram definidos KPIs relevantes para o negócio e visualizações dinâmicas com paginação para facilitar a leitura e tornar a análise mais interativa. A valorização da leitura em Z foi aplicada nas visualizações, a fim de direcionar o olhar do usuário para as informações mais importantes. O resultado foi um dashboard completo e intuitivo, que permite a tomada de decisão mais assertiva e ágil. Vale ressaltar que as atualizações de dados são realizadas de forma automatizada através do Gateway de Dados do Power BI, garantindo a consistência e precisão das informações apresentadas.


<img src="https://user-images.githubusercontent.com/66785163/236552159-d7b5d928-ad72-4def-880d-fd6852b0e34f.png" alt="Star Schema" width="1000"/>

<p align="center"><b>Exemplo do modelo Star Schema</b></p>

## Principais KPIs

- Faturamento: É a quantidade de dinheiro que a concessionária recebe com a venda de veículos mas não necessariamente é o valor que ela já recebeu como na receita. No projeto o faturamento pode ser avaliada em relação ao tempo (ano e mês), por veículo, por loja e estados.
- Devoluções: É a quantidade de valor(monetário) devolvido e tambem a quantidade unitária desses veículos devolvidos  em relação ao tempo (ano e mês), por veículo e estados
- Unidades vendidas: É a quantidade de veículos que a concessionária vendeu em um determinado período. Esse indicador é importante para avaliar o desempenho das vendas e para ajudar a prever a demanda futura.
- Ticket Médio: É valor médio de faturamento da empresa por unidade vendida de um determinado tipo de veículo.
- Faturamento x Meta: Esse indicador compara o faturamento da concessionária com a meta determinada, nesse caso a meta é definida por mês.
- % Meta: Ela mostra a % alcançada da meta em relação ao faturamento em relação ao tempo (ano e mês)
- % Devoluções: Mostra em % a quantidade de veículos devolvidos, é possível avaliar por veículo,  em relação ao tempo (ano e mês), por marca e por estados


# UI/UX e design

Para a construção do design do dashboard no Power BI, utilizei o Figma, uma plataforma de design que possibilitou a criação de um layout moderno e atrativo para os usuários. O Figma permitiu uma fácil edição e manipulação dos elementos visuais, como gráficos, cores e ícones, o que resultou em uma apresentação mais coesa e profissional dos dados. Com o Figma, foi possível criar protótipos interativos do dashboard, testando diferentes layouts e elementos visuais, antes de implementá-los no Power BI, o que reduziu significativamente o tempo de desenvolvimento e melhorou a qualidade do produto final.

[![Figma](https://user-images.githubusercontent.com/66785163/236950034-46d5f753-622c-45e4-9731-3e6c42c9375d.png "Figma")](https://www.figma.com/file/zLawqWlCkYPSW3Z0fvLn27/Dashboard-Concession%C3%A1ria?type=design&node-id=2-14&t=BkpZswiwXGXlRiwa-0 "Acesse o Figma completo")
<p align="center"><b>Clique na imagem para o Figma completo</b></p>

# Gestão do Projeto

No projeto de Business Intelligence utilizei o Trello para gerenciar as tarefas, com User History e Critérios de Aceitação. Para isso, adotei a metodologia ágil Scrum, permitindo a adaptação às necessidades do negócio e do mercado. O Scrum garantiu o alinhamento do projeto com os requisitos do cliente e possibilitou a definição e priorização de tarefas com base nas necessidades e objetivos do negócio.


- A coluna "Backlog" foi  usada para listar todas as tarefas que precisam ser concluídas durante o projeto, classificando-as por ordem de prioridade.
- A coluna "Sprint Backlog" foi usada para listar as tarefas selecionadas para serem concluídas durante o sprint atual.
- A coluna "Doing" foi usada para listar as tarefas em andamento pela equipe.
- A coluna "Review" foi usada para listar as tarefas concluídas que precisam ser revisadas antes de serem marcadas como concluídas.
- A coluna "Done" foi usada para listar as tarefas que foram concluídas e revisadas, prontas para serem entregues ou colocadas em produção.

<img src="https://user-images.githubusercontent.com/66785163/236954748-fa301815-ef6c-4c4b-915e-de33cef6c4ae.png" alt="Trello" width="1000"/>


# Entrega de valor

 A solução permitiu aos gestores acesso em tempo hábil a informações estratégicas sobre desempenho de modelos, preferências dos clientes, taxas de devolução e outros KPIs importantes para o negócio.  Isso incluiu benefícios como tomada de decisões mais rápidas e assertivas, automação de processos, segurança e qualidade dos dados, economia em custos de infraestrutura e melhoria da experiência do cliente através da análises de preferências de veículos mais vendidos e menos devolvidos, sendo possível focar mais no que o cliente quer e alcançar uma maior lucratividade, além de poder analiser os KPIs ao longo do tempo e fazer comparativos ano a ano.
 
<p align="center"><a href="https://app.powerbi.com/view?r=eyJrIjoiYzI0MTUxOGQtY2E2OS00NjU3LTkxNDUtOGEzOTE3YmEyNzg1IiwidCI6IjNmMWEwN2Q1LWViY2ItNGM4ZC04NWM4LTZmNWFmZTMwMjc2ZCJ9" style="font-size:20px;">Clique aqui para acessar o dashboard</a></p>

#Fontes de inspiração para desenvolvimento do projeto

https://angelosalton.com.br/data-lake-simples-aws.html
https://medium.com/@aliatakan/creating-a-data-lake-by-using-aws-s3-glue-and-athena-86e43738d8f2
https://arrudaconsulting.com.br/treinamento-pentaho-aws/
https://aws.amazon.com/blogs/big-data/enrich-datasets-for-descriptive-analytics-with-aws-glue-databrew/
