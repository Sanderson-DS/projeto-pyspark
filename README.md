
# 🧠 Projeto PySpark — Ambiente Local (Windows)

Este projeto utiliza Apache Spark + PySpark + Jupyter Notebook + VS Code em ambiente local no Windows.

Este guia documenta a configuração completa das ferramentas para desenvolvimento e estudo com Spark local.

---

# 📦 Pré-requisitos

- Windows 10/11
- Python 3.11+
- VS Code
- Extensões VS Code:
  - Python
  - Jupyter

---

# ☕ 1) Instalação do JDK (Java)

1. Instale o JDK (exemplo):
   C:\Java\jdk-25.0.2

2. Configure as variáveis de ambiente:

JAVA_HOME
C:\Java\jdk-25.0.2

Adicionar ao PATH:
C:\Java\jdk-25.0.2\bin

3. Teste no terminal:

java -version

---

# ⚡ 2) Instalação do Apache Spark

1. Extraia o Spark em:
C:\Spark

2. Configure as variáveis de ambiente:

SPARK_HOME
C:\Spark

Adicionar ao PATH:
C:\Spark\bin

3. Teste:

spark-shell --version
spark-submit --version

---

# 🐘 3) Hadoop (Winutils) para Windows

Necessário para evitar erros de permissão no Windows ao usar Spark.

1. Criar diretório:
C:\hadoop\bin

2. Baixar winutils.exe (Windows 64 bits)
Repositório:
https://github.com/cdarlint/winutils

3. Colocar em:
C:\hadoop\bin\winutils.exe

4. Configurar variável:

HADOOP_HOME
C:\hadoop

Adicionar ao PATH:
C:\hadoop\bin

---

# Criar diretórios temporários

Criar:

C:\tmp\hive  
C:\tmp\spark

Permissões (PowerShell):

C:\hadoop\bin\winutils.exe chmod -R 777 C:\tmp\hive  
C:\hadoop\bin\winutils.exe chmod -R 777 C:\tmp\spark  

---

# 🐍 4) Criar Ambiente Virtual (venv)

Dentro da pasta do projeto:

python -m venv venv

Ativar (Git Bash):

source venv/Scripts/activate

Instalar dependências:

python -m pip install --upgrade pip  
pip install pyspark ipykernel jupyter  

Registrar kernel:

python -m ipykernel install --user --name "pyspark-venv" --display-name "PySpark (venv)"

---

# 🧩 5) Configurar VS Code (.env)

Criar arquivo .env na raiz do projeto:

JAVA_HOME=C:\Java\jdk-25.0.2  
SPARK_HOME=C:\Spark  
HADOOP_HOME=C:\hadoop  
PYSPARK_PYTHON=C:\CAMINHO\DO\PROJETO\venv\Scripts\python.exe  
SPARK_LOCAL_DIRS=C:\tmp\spark  

Criar .vscode/settings.json:

{
  "python.envFile": "${workspaceFolder}/.env"
}

Reiniciar VS Code.

Selecionar kernel:
PySpark (venv)

---

# 🚀 6) Configuração Recomendada do SparkSession (Windows-safe)

Utilize esta configuração base nos notebooks:

import os  
from pyspark.sql import SparkSession  

os.makedirs(r"C:\tmp\spark", exist_ok=True)  
os.makedirs(r"C:\tmp\hive", exist_ok=True)  

spark = (  
    SparkSession.builder  
    .master("local[*]")  
    .appName("PySpark-Project")  
    .config("spark.ui.enabled", "true")  
    .config("spark.ui.host", "127.0.0.1")  
    .config("spark.driver.host", "127.0.0.1")  
    .config("spark.driver.bindAddress", "127.0.0.1")  
    .config("spark.local.dir", r"C:\tmp\spark")  
    .config("spark.sql.warehouse.dir", r"C:\tmp\hive")  
    .config("spark.sql.shuffle.partitions", "16")  
    .config("spark.default.parallelism", "16")  
    .config("spark.driver.memory", "2g")  
    .config("spark.executor.memory", "2g")  
    .getOrCreate()  
)  

print("Spark version:", spark.version)  
print("Spark UI:", spark.sparkContext.uiWebUrl)  

---

# 🌐 Spark UI

Abrir no navegador:

http://127.0.0.1:4040  

Se estiver ocupada:

http://127.0.0.1:4041  
http://127.0.0.1:4042  

---

# 🧪 Teste de Job (gera Jobs/Stages)

df = spark.range(0, 20000000).repartition(16)  
df.groupBy((df.id % 1000).alias("k")).count().count()  

Se necessário reduzir volume:

df = spark.range(0, 5000000).repartition(8)  
df.groupBy((df.id % 1000).alias("k")).count().count()  

---

# 🛑 Encerrar Sessão

spark.stop()

---

# ⚠️ Problemas Comuns

Spark UI não abre:
- Verifique se Spark está rodando
- Não execute spark.stop() antes de abrir a UI
- Verifique portas 4040, 4041, 4042

Erro WinError 10054:
- Reduzir volume do job
- Ajustar memória
- Confirmar diretórios temp
- Confirmar kernel correto (venv, não conda)

---

# ✅ Checklist Final

No terminal:

python -c "import sys; print(sys.executable)"  
python -c "import pyspark; print(pyspark.__version__)"  
java -version  
spark-submit --version  

No notebook:

import os, sys  
print("Python:", sys.executable)  
print("JAVA_HOME:", os.getenv("JAVA_HOME"))  
print("SPARK_HOME:", os.getenv("SPARK_HOME"))  
print("HADOOP_HOME:", os.getenv("HADOOP_HOME"))  

---

# 📓 Notebooks

A pasta `notebooks/` contém os notebooks utilizados durante o curso para demonstrar os principais conceitos do PySpark.  
Cada arquivo segue a mesma estrutura: cria (ou reutiliza) uma `SparkSession`, executa alguns exemplos e, ao final, encerra a sessão.

| Notebook | Objetivo / Conteúdo |
|----------|---------------------|
| **script1_rdd.ipynb** | **Introdução a RDDs** <br>– cria `SparkSession` e `SparkContext`;<br>– paraleliza listas; conta, coleta e imprime dados;<br>– lê arquivo CSV (`../data/food_coded.csv`) como RDD e faz operações `first()`, `count()` etc.;<br>– salva fragmentos em `../output/`.<br>É o ponto de partida para entender a API RDD. |
| **script2_rdd_text_miner.ipynb** | **Text mining com RDDs** <br>– carrega um texto (regiões Norte/Sul) em um RDD;<br>– separa palavras (`flatMap`, `split`), normaliza (`lower`), filtra *stopwords*;<br>– conta ocorrências (`map`/`reduceByKey`);<br>– imprime resultado ordenado; `<br>– mostra `toDebugString()` do plano. |
| **script3_rdd_joins.ipynb** | **Operações de join em RDDs** <br>– dois arquivos com dados (profissionais e salários);<br>– transforma em pares `(chave, valor)`;<br>– executa `join`, `leftOuterJoin`, `rightOuterJoin`, etc.;<br>– demonstra coleta de resultados. |
| **script4_dataframe.ipynb** | **Introdução a DataFrames** <br>– cria DataFrame simples (cidade, população, time, data);<br>– mostra `show()`, `printSchema()`, `describe()`; <br>– operações de seleção, filtro, `withColumn` usando funções (`upper`);<br>– exemplifica agregações e coleta. |
| **script5_dataframe_arquivos.ipynb** | **Leitura e limpeza de CSV em DataFrame** <br>– lê `../data/food_coded.csv` com `header=true`; <br>– examina schema, mostra primeiras linhas;<br>– filtra valores inconsistentes (`none`, `nan`);<br>– exemplos de agrupamentos, agregações e transformações diversas;<br>– encerra o `spark.stop()` no final. |
| **script6_sparksql.ipynb** | **Spark SQL** <br>– registra DataFrames como views temporárias;<br>– executa consultas SQL (incluindo joins, `group by` etc.);<br>– apresenta resultados em tabelas HTML renderizadas pelo notebook. |
| **script7_json_sql.ipynb** | **JSON + SQL** <br>– carrega arquivo JSON de sentimentos (`../data/sentimento.json`);<br>– inspeciona schema com `printSchema()`;<br>– explode/seleciona campos aninhados (`qas`, `id`, `question`);<br>– cria temp view e filtra registros negativos. |
| **script8_streaming.ipynb** | **Spark Streaming clássico** <br>– demontra leitura de socket TCP na porta 9999;<br>– configura `StreamingContext` e processa linhas em tempo real;<br>– exibe resultados e discute configuração da fonte (nc/ncat). |
| **script9_streaming_structured.ipynb** | **Structured Streaming** <br>– prepara `SparkSession` (separado da criação no cell 9);<br>– define stream de socket → transforma linhas em palavras (`explode`/`split`);<br>– conta palavras de forma contínua e grava em **memory sink** (`queryName("wc")`);<br>– run/visualiza query periodicamente, mostra como parar as queries ativas.<br>– checklist final de funcionamento (nc rodando, notebooks executados). |
| **teste_ambiente.ipynb** | **Validações e ajustes do SparkSession** <br>– constrói sessão com várias configurações (UI, dirs temporários, paralelismo, limites de memória);<br>– imprime variáveis de ambiente (`JAVA_HOME`, `SPARK_HOME`, …) e versão do Spark;<br>– executa um pequeno job (`spark.range(10).show()`);<br>– visualiza `sparkContext.uiWebUrl` e faz testes de UI;– encerra com `spark.stop()`. |

> ⚠ **Nota:** os notebooks foram criados com kernel **PySpark (venv)** e dependem das pastas `data/` e `output/` no nível raiz.

## Como usar

1. Crie e ative o ambiente virtual conforme descrito no README principal.
2. Registre o kernel Jupyter (`python -m ipykernel install …`).
3. Abra o notebook desejado no VS Code e execute as células sequencialmente.
4. Para os exemplos de streaming (portas TCP 9999) use `nc -lk 9999` no terminal (WSL/MobaXterm).

## Estrutura dos notebooks

Cada notebook contém:

- **Markdown** com explicações e objetivos.
- **Células de código** que importam `pyspark.sql.SparkSession` ou `pyspark.streaming.StreamingContext`.
- Várias leituras de arquivo em `../data/` ou uso de coleções internas.
- Exibições (`show()`, prints) dos resultados no notebook.
- Comentários em Português explicando os passos.
- Finalização com `spark.stop()` quando apropriado.

---

Esse README serve tanto como roteiro de estudo quanto como documentação do que cada notebook faz.  
Basta navegar pelos arquivos na pasta e consultá‑los para executar ou adaptar os exemplos ao seu próprio uso.