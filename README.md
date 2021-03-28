# Web-Scraping com Beautiful Soup

---

### O que é Web Scrapping?

Scraping (raspagem em tradução literal) é o processo de extração, cópia e triagem de dados. Quando extraímos dados da web (de sites, páginas, etc.), nós denominamos tal tarefa como ***web-scraping***. 

O web-scraping permite aos desenvolvedores coletar e analisar os dados da internet. A partir dele é possível automatizar a maioria das tarefas que um humano faz enquanto navega pelo browser.

Pode ser utilizado de diferentes formas: dados para pesquisa, análise de conteúdo, comparação de preços e popularidade, monitoramento, vendas e marketing, etc.

---

### Beautiful Soup

![Ícone Python e Beautiful Soup](https://d33wubrfki0l68.cloudfront.net/da1e0318622d0d2134f72714d248a339780799af/1be46/blog/python-web-scraping-beautiful-soup/beautiful_soup.png)
É uma biblioteca em Python que nos permite extrair dados de documentos HTML e XML. Geralmente os dados estão desorganizados e/ou mal estruturados, então, utilizando essa biblioteca, podemos organizar tais dados, remover as informações desnecessárias e estruturá-las para que fiquem apresentáveis, entendíveis e mais fáceis de analisá-las e extrair informações delas.

---

### Quickstart

Vamos começar acessando uma página web e extraindo o título dela.

> A fim de tornar o tutorial mais dinâmico, não faremos nenhuma instalação local. Utilizaremos o [Repl.it](https://replit.com/).


```python
# Importa as ferramentas necessárias
import requests

from bs4 import BeautifulSoup

# Define a url que iremos acessar
url = 'https://scrapethissite.com/pages/'

# Faz a requisição para a url
response = requests.get(url)

# Instancia o conteúdo retornado como um objeto do Beautiful Soup
soup = BeautifulSoup(response.text, 'html.parser')

print(soup.title.text)

# Resposta esperada
Learn Web Scraping | Scrape This Site | A public sandbox for learning web scraping
```

Vamos agora extrair todas as URLs da página.

```python
# Busca todos as tags <a> no HTML
links = soup.find_all('a')
for link in links:
    print(link.get('href'))

# Resposta esperada
/
/pages/
/lessons/
/faq/
/login/
/pages/simple/
/pages/forms/
/pages/ajax-javascript/
/pages/frames/
/pages/advanced/
'''
```

Nesse site, eles só disponibilizam os endpoints no atributo href. Para exibir o link completo, vamos fazer uma modificação, concatenando a url do site com o endpoint retornado.

```python
for link in links:
    print(f"https://scrapethissite.com{link.get('href')}")

# Resposta esperada
https://scrapethissite.com/
https://scrapethissite.com/pages/
https://scrapethissite.com/lessons/
https://scrapethissite.com/faq/
https://scrapethissite.com/login/
https://scrapethissite.com/pages/simple/
https://scrapethissite.com/pages/forms/
https://scrapethissite.com/pages/ajax-javascript/
https://scrapethissite.com/pages/frames/
https://scrapethissite.com/pages/advanced/
```

### Estrutura do HTML

![Estrutura do HTML](https://www.tutorialspoint.com/beautiful_soup/images/html_tree_structure.jpg)

O elemento raiz da árvore é o **HTML**, que pode ter nós pais, filhos e irmãos, e isso será determinado pela sua posição na estrutura. Para mover-se entre os elementos, atributos, textos, etc., você deve mover-se entre os nós.

### Extraindo dados do site da Gol

Iremos agora acessar o site da gol via scraping e coletar as ofertas de vôos na região Nordeste.

```python
import requests

from bs4 import BeautifulSoup

# Cabeçalhos necessários para que os sites mais seguros permitam que nós façamos scraping neles
HEADERS = ({
    'User-Agent':
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36',
    'Accept-Language': 'en-US, en;q=0.5'
})

url = 'https://www.voegol.com.br/pt/sua-viagem/ofertas/nordeste'
response = requests.get(url, headers=HEADERS)

soup = BeautifulSoup(response.text, 'html.parser')

# Busca todos os cards da página
cards = soup.find_all('div',
                      attrs={'class': 'col-lg-4 col-md-4 col-xs-12'})

for card in cards:
    origin, destination = card.find_all('span',
                                        attrs={'class': 'preto fs18'})
    price = card.find('strong', attrs={'class': 'fs32'})

    # Trata o price, que vem em um formato não tão agradável visualmente
    price = price.text.replace(' ', '')
    price = price.replace('\r\n', '')
    
    print('origin:', origin.text)
    print('destination:', destination.text)
    print('price:', price)
    print('------------------------------')

# Resposta esperada
origin: Salvador (SSA)
destination: João Pessoa (JPA)
price: R$212,03
------------------------------
origin: Salvador (SSA)
destination: Teresina (THE)
price: R$182,03
------------------------------
origin: Teresina (THE)
destination: Salvador (SSA)
price: R$177,06
------------------------------
origin: Salvador (SSA)
destination: São Luís (SLZ)
price: R$192,03
------------------------------
origin: João Pessoa (JPA)
destination: Salvador (SSA)
price: R$194,47
------------------------------
```

### Salvando os dados em uma planilha

Adicionaremos algumas linhas para a criação do arquivo e a inserção dos dados.

```python
import csv
import requests

from bs4 import BeautifulSoup

HEADERS = ({
    'User-Agent':
    'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36',
    'Accept-Language': 'en-US, en;q=0.5'
})

# Cria o arquivo csv
file = open('voos.csv', 'w')
file = csv.writer(file)

# Define o cabeçalho
file.writerow(['Origem', 'Destino', 'Preço'])

url = 'https://www.voegol.com.br/pt/sua-viagem/ofertas/nordeste'
response = requests.get(url, headers=HEADERS)

soup = BeautifulSoup(response.text, 'html.parser')

cards = soup.find_all('div', attrs={'class': 'col-lg-4 col-md-4 col-xs-12'})

for card in cards:
    origin, destination = card.find_all('span', attrs={'class': 'preto fs18'})
    price = card.find('strong', attrs={'class': 'fs32'})
    price = price.text.replace(' ', '')
    price = price.replace('\r\n', '')

    # Adiciona uma linha de dados no arquivo csv
    file.writerow([origin.text, destination.text, price])
```
