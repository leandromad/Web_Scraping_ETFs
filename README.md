# Web_Scraping_ETFs
Dados financeiros de todos os ETFs do MUNDO com Python.

## Notas
O arquivo `requirements.txt`  contem todas as bibliotecas Python que foram utilizadas no notebook, para instalar essas bibliotecas basta abrir o terminal e indicar a pasta do projeto, e então no terminal inserir o comando:
```
pip install -r requirements.txt
```

### O Projeto
Este projeto tem como objetivo fazer raspagem de dados (web scraping) do site https://www.etf.com para obter dados de ETF (Exchange Traded Fund) do mundo todo e analisar quais ETF obtiveram a melhor performance comparando os ultimos 3, 5 e 10 anos.
Ao final obtemos um dataframe com os 10 melhores ETFs:

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/218269533-594f5715-c9ef-46cc-9b47-c5fa8a40bee6.png" />
  <p> Figura 1 - 10 melhores ETFs</p>
</div>

As bibliotecas utilizadas:

```
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from webdriver_manager.chrome import ChromeDriverManager
import time
import pandas as pd
```

Inicialmente devemos abrir a pagina que será realizada o scraping. Conforme abaixo:

```
driver = webdriver.Chrome(service = Service(ChromeDriverManager().install()))
url = "https://www.etf.com/etfanalytics/etf-finder"
driver.get(url)
```
Nesse caso, estamos instanciando o navegador como Chrome, caso fosse preferível a utilização de outro navegador a mudança deverá ser feita conforme a documentação.  [Documentation](https://pypi.org/project/webdriver-manager/)

O site que queremos acessar é passado para a varíavel ``url`` e chamamos o navegador.

Ao abrir o site, nos deparamos com a seguinte tabela:

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219144018-32817977-7f12-4aad-b481-2c523f64fde9.png" />
  <p> Figura 2 - Tabela do site. </p>
</div>

Para a raspagem de dados (scraping) ocorrer de maneira mais rapida vamos selecionar a visualização de 100 itens que se encontra abaixo da tabela. Isso deve ser feito utilizando o selenium. O script abaixo faz a seleção do botão.

```
time.sleep(5)
botao_100 = driver.find_element("xpath", '''html/body/div[5]/section/div/div[3]/section/div/div/div/div/div[2]/section[2]/div[2]
/section[2]/div[1]/div/div[4]/button/label/span''')

driver.execute_script('arguments[0].click();', botao_100)
``` 
Utilizamos o time.sleep(5) para dar um tempo de 5 segundos na página para carregar o elemento, e passados o full xpath do elemento na váriavel ``botao_100``. O comando driver.execute_script é executado para clickar no botão.

A próxima etapa é obter o total de páginas que temos com esse estilo de visualização. 

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219146324-be47240f-2db2-45dd-a836-21176f353a78.png" />
  <p> Figura 3 - Quantidade de páginas.</p>
</div>

Temos na figura o total de páginas localizado á direita. Podemos capturar a quantidade de páginas através do seu full xpath e fazer os tratamentos necessários conforme script abaixo:

```
numero_paginas = driver.find_element("xpath", '''html/body/div[5]/section/div/div[3]/section/div/div/div/div/div[2]
/section[2]/div[2]/section[2]/div[2]/div/label[2]''')
numero_paginas = int(numero_paginas.text.replace("of ",""))
numero_paginas
```

Com o número de páginas obtido podemos percorrer o loop para adquirir todas as tabelas: 

```
lista_de_tabela_por_pagina = []

for pagina in range(0, numero_paginas):
    
    tabela = driver.find_element("xpath", '''/html/body/div[5]/section/div/div[3]
    /section/div/div/div/div/div[2]/section[2]/div[2]/div/table''')

    html_tabela = tabela.get_attribute("outerHTML")

    tabela_final = pd.read_html(html_tabela)[0]

    lista_de_tabela_por_pagina.append(tabela_final)
    
    botao_avancar_pagina = driver.find_element("xpath", "/html/body/div[5]/section/div/div[3]/section/div/div/div/div/div[2]/section[2]/div[2]/section[2]/div[2]/div/span[2]")
    
    driver.execute_script("arguments[0].click();", botao_avancar_pagina)
    
base_de_dados_completa = pd.concat(lista_de_tabela_por_pagina)

base_de_dados_completa
```
Para cada uma das páginas existente vamos guardar a tabela em uma lista ``lista_de_tabela_por_pagina``, após iremos avançar para a próxima página ``botao_avanca_pagina`` e assim sucessivamente, ao atingir o ``numero_paginas`` chegamos ao fim do loop e temos nosso dataframe concatenando as listas ``base_de_dados_completa = pd.concat(lista_de_tabela_por_pagina)``

A base de dados nesse momento é:

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219215424-bec9786a-c9a4-4579-9241-da3e70deb5fe.png" />
  <p> Figura 4 - Base de dados 1.</p>
</div>

Como no final queremos classificar os ETFs conforme sua performance temos que adiquirir os dados de performance, existe uma tabela própria para isso. Iremos captura-la do próprio site:

```
botao_performance = driver.find_element("xpath", '''/html/body/div[5]/section/div/div[3]/section/div/div/div/div/div[2]
/section[2]/div[2]/ul/li[2]/span''')

driver.execute_script("arguments[0].click();", botao_performance)

for pagina in range (0, numero_paginas):
    
    botao_voltar = driver.find_element("xpath", '''/html/body/div[5]/section/div/div[3]/section/div/div/div/div/div[2]
    /section[2]/div[2]/section[2]/div[2]/div/span[1]''')
    
    driver.execute_script("arguments[0].click();", botao_voltar)
```

A primeira parte do script ``botao_performance`` localiza o full xpath do botão que encaminha para a tabela performance, e fazemos a execução do click em ``driver.execute_script("arguments[0].click();", botao_performance)``. Como nesse momento estamos na última página por conta do último loop executado, executamos outro loop para voltar a página 1.

E agora, da mesma maneira que adquirimos os dados para o Dataframe anterior fazemos para esse Dataframe.
```
lista_de_tabela_por_pagina = []

for pagina in range(0, numero_paginas):
    
    tabela = driver.find_element("xpath", '''/html/body/div[5]/section/div/div[3]
    /section/div/div/div/div/div[2]/section[2]/div[2]/div/table''')

    html_tabela = tabela.get_attribute("outerHTML")

    tabela_final = pd.read_html(html_tabela)[0]

    lista_de_tabela_por_pagina.append(tabela_final)
    
    botao_avancar_pagina = driver.find_element("xpath", "/html/body/div[5]/section/div/div[3]/section/div/div/div/div/div[2]/section[2]/div[2]/section[2]/div[2]/div/span[2]")
    
    driver.execute_script("arguments[0].click();", botao_avancar_pagina)
    
base_de_dados_performance = pd.concat(lista_de_tabela_por_pagina)

base_de_dados_performance
```
E agora temos o seguinte dataframe:

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219216663-3fd28585-38a8-4402-9ff8-4a1b05512e33.png" />
  <p> Figura 5 - Dataframe 2.</p>
</div>

Para o Dataframe 1 da Figura 4 iremos passar o Ticker como index:
```
base_de_dados_completa = base_de_dados_completa.set_index("Ticker")

base_de_dados_completa
``` 

Agora temos o Dataframe 1 da Figura 4 transformado em:

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219640800-ece87aa6-edf8-4c4d-a799-b58497c65165.png" />
  <p> Figura 6 - Dataframe 1 transformado.</p>
</div>

O mesmo iremos fazer para o Dataframe 2 da Figura 5 e também iremos selecionar as colunas ``3 Years','5 Years','10 Years`` desse Dataframe.

```
base_de_dados_performance = base_de_dados_performance.set_index("Ticker")
base_de_dados_performance = base_de_dados_performance[['3 Years','5 Years','10 Years']]
base_de_dados_performance
```

E obtemos o seguinte Dataframe

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219641066-475c9534-73ad-40b5-bf3c-015fa4cbc8d1.png" />
  <p> Figura 6 - Dataframe 2 transformado.</p>
</div>

Ainda temos que tratar esse Dataframe, primeiro iremos substituir os dados faltantes ``--`` por ``NaN``.

```
base_de_dados_performance = base_de_dados_performance.replace("--",pd.NA)

base_de_dados_performance

```
E agora vamos dropar os ``NaN`` do dataframe:
``` 
base_de_dados_performance = base_de_dados_performance.dropna()

base_de_dados_performance
```

E o nosso Dataframe 2 de performance agora é:

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219643880-d74d8e26-eb0e-4dde-98f3-296b0d046c7b.png" />
  <p> Figura 7 - Dataframe 2 transformado 2.</p>
</div>

Como os dados desse dataframe estão em porcentagem e formato string iremos passar para float:

```
for i in base_de_dados_performance.columns:

    base_de_dados_performance[i] = base_de_dados_performance[i].str.rstrip('%').astype(float)/100

base_de_dados_performance
```

E então podemos unir os dois dataframes:
```
base_de_dados_final = base_de_dados_completa.join(base_de_dados_performance, how = "inner")
base_de_dados_final
```

Temos o seguinte Dataframe:

<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219645039-449ce081-9cea-4b1c-be2b-f6e1b5f13687.png" />
  <p> Figura 8 - Base de dados final.</p>
</div>

Ainda queremos tirar dos nossos dados ETFs que estão alavancados portanto utilizaremos um filtro:
```
base_de_dados_final = base_de_dados_final[~base_de_dados_final['Segment'].str.contains("Leveraged")]
base_de_dados_final
``` 

Podemos agora fazer nosso rank dos ultimos 3, 5 e 10 anos.
```
base_de_dados_final['rank_3_anos'] = base_de_dados_final['3 Years'].rank(ascending = False)
base_de_dados_final['rank_5_anos'] = base_de_dados_final['5 Years'].rank(ascending = False)
base_de_dados_final['rank_10_anos'] = base_de_dados_final['10 Years'].rank(ascending = False)

base_de_dados_final
```

E fazer um rank final com a soma desses rank's:
```
base_de_dados_final['rank_final'] = base_de_dados_final['rank_3_anos'] + base_de_dados_final['rank_5_anos'] + base_de_dados_final['rank_10_anos'] 
base_de_dados_final
```

Por fim vamos ordenar esse dataframe do menor rank para o maior
```
melhores_etfs = base_de_dados_final.sort_values(by = 'rank_final')
melhores_etfs
```

E mostramos os 10 melhores ETFs em quesito performance nos ultimos anos.
```
melhores_etfs.head(10)
```


<div align="center">
  <img src="https://user-images.githubusercontent.com/82683162/219645993-bd5a3665-025d-443e-8cfe-bc1b4f5c7d87.png" />
  <p> Figura 9 - Melhores ETFs.</p>
</div>
