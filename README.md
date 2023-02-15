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

