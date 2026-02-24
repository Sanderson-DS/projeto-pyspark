
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