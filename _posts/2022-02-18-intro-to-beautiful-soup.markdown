---
layout: post
title:  "Extração de dados com python (web scraping com BeautifulSoup)"
date:   2022-02-18 19:31 -0300
categories: [python, webscraping]
---

Beautiful Soup é uma biblioteca python para extração de dados de arquivos HTML e XML.

O que o Beautiful Soup faz é basicamente transformar estes documentos em objetos python. Os principais objetos Beautiful Soup são: 

<ul>
    <li>
        <strong>Tag</strong>: objeto referente às tags html
    </li>
    <li>
        <strong>NavigableString</strong>: uma classe que contém as strings que existem dentro das tags html
    </li>
    <li>
        <strong>BeautifulSoup</strong>: objeto referente ao documento do qual a extração será feita
    </li>
    <li>
        <strong>Comment</strong>: um objeto <em>NavigableString</em> especial usado para tornar o código mais legível
    </li>
</ul>

Você pode instalar o Beautiful Soup com o<a href="https://pypi.org/project/pip" target="_blank"> instalador de pacotes</a> do python:

```bash
$ pip install beautifulsoup4
```


Para fazer requisições http, vou usar a biblioteca Requests:

```bash
$ python -m pip install requests
```

Neste exemplo, irei extrair dados de uma página do website <a href="https://www.adam4eve.eu/" target="_blank">Adam4Eve</a>.


Com todas as bibliotecas instaladas, começo o meu script python fazendo as importações:

{% highlight python %}
from bs4 import BeautifulSoup
import requests
{% endhighlight %}


E então, escrevo as linhas:

{% highlight python %}
url = "https://www.adam4eve.eu/commodity.php?region=10000002&typeID=46375"
result = requests.get(url)
soup = BeautifulSoup(result.content, 'html.parser')
print(soup.prettify())
{% endhighlight %}

Na primeira linha, eu crio uma variável chamada "url" e informo a página da web que eu quero fazer o <i>web scraping</i>.

Logo em seguida, faço a requisição http com o Requests, depois passo o resultado da requisição ao construtor do BeautifulSoup e informo o <i>parser</i> que eu quero usar, que neste caso será o <code>'hml.parser'</code>.

A  linha <code>print(soup.prettify())</code>, faz uso do método <code>prettify()</code> para exibir o corpo inteiro do documento html.

O que eu quero agora é extrair dados de uma parte específica da página: uma tabela com informações de um determinado item.

![printscrap]({{"/img/marketdetails.png" | absolute_url }})

Observe a seguinte função:

{% highlight python %}
def scraper():
    sectioncontent = soup.find('table', class_='no_border')
    print(sectioncontent)
{% endhighlight %}

Na função acima, o método <code>find()</code> está sendo usado para encontrar o elemento <i><b>table</b></i>. Explicarei esse método logo à frente. 

Na passagem de parâmetros, <code>'table'</code> é referente ao elemento <em>table</em> e <code>class_</code>, ao atributo html <em>class</em>. 

A saída desta função no console mostra o elemento <i><b>table</b></i> e todos os seus descendentes:

{% highlight html %}
    <table class="no_border">
    <tr><td style="vertical-align: top">
    <table style="background-color: white; border: 1px solid darkgrey; font-size: 0.8em">
    <tr>
    <td colspan="2" style="font-size: 20px; padding-right: 25px; padding-left: 10px; white-space: nowrap">Daily Alpha Injector</td>
    <td rowspan="2"><img src="https://image.eveonline.com/Type/46375_64.png" style="border: 1px solid black; padding: 1px"/></td>
    <td rowspan="2">
    <table style="margin-right: 20px">
    <tr><td class="pad3 bold">TypeID:</td><td class="pad3 right">46375</td></tr>
    <tr><td class="pad3 bold">Published:</td><td class="pad3 right" style="color: green">Yes</td></tr>
    <tr><td class="pad3 bold">Volume</td><td class="pad3 right">0,01 m³</td></tr>
    <tr><td class="pad3 bold">Mass</td><td class="pad3 right">0,00 kg</td></tr>
    </table>
    </td><td rowspan="2" style="vertical-align: top; padding-right: 20px">
    <table>
    <tr><td class="pad3 bold">Tradeable:</td><td class="pad3" style="color: green">Yes<a 1="" href="market_orders.php?typeID=46375"> <img src="templates/img/table.png" title="Show market orders"/></a></td><td><span name="1"><a href="market_orders.php?mgid=1922">Pilot's Services</a> / <a href="market_orders.php?mgid=2358">Skill Trading</a></span></td></tr>
    <tr><td class="pad3 bold vtop">Buildable:</td><td class="pad3 vtop" style="color: red">No </td><td></td></tr>
    <tr><td class="pad3 bold vtop">Inventable:</td><td class="pad3 vtop" style="color: red">No</td><td></td></tr>
    </table>
    </td>
    </tr>
    <tr><td style="padding-left: 10px; padding-right: 10px">
    <table>
    <tr>
    <td class="pad3 bold">Group:</td><td class="pad3"><a href="price_chart.php?groupID=1739">Skill Injectors</a></td>
    <td><a href="price_chart.php?groupID=1739"><img alt="Chart" src="templates/img/chart_curve.png" title="Goto price history charts for group"/></a></td>
    </tr>
    <tr>
    <td class="pad3 bold">Category:</td><td class="pad3"><a href="price_chart.php?catID=5">Accessories</a></td>
    <td><a href="price_chart.php?catID=5"><img alt="Chart" src="templates/img/chart_curve.png" title="Goto price history charts for category"/></a></td>
    </tr>
    </table>
    </td><td style="padding-right: 10px">
    <table>
    <tr><td class="pad3 bold">MetaLevel:</td><td class="pad3">0</td></tr>
    <tr><td class="pad3 bold">MetaGroup:</td><td class="pad3">Tech I</td></tr>
    </table>
    </td></tr>
    </table>
    </td></tr></table>
{% endhighlight %}


### find() e find_all()

Os métodos <i><strong>find()</strong></i> e <i><b>find_all()</b></i>, são métodos usados para buscas na árvore de análise do documento.

<i><strong>find()</strong></i>, faz uma busca entre os elementos e devolve o primeiro elemento que é equivalente ao parâmetro passado no método. Se a seção que será analisada só contiver um único elemento <i><b>title</b></i>, a instrução <code>find('title')</code>, poderá ser usada. Por outro lado, se houver mais de um elemento <i><b>title</b></i>, o método também poderá ser usado, mas retornará apenas o primeiro elemento <i><b>title</b></i> que encontrar.

<i><b>find_all()</b></i>, faz uma busca pelos elementos e devolve <strong>todos</strong> os descendentes da tag que corresponde ao filtro passado como parâmetro. <code>find_all('a')</code>, por exemplo, retorna todos os elementos âncora em questão.

Ok, fizemos a extração da tabela. Mas isso ainda não é exatamente o que queremos. As tags html de um documento têm a função de informar ao navegador como o conteúdo que está dentro delas deve ser interpretado. Para o que queremos agora, no entanto, elas não terão uso algum. O que realmente nos interesssa é o texto que está dentro destas tags.

### .string e get_text()

Como já foi dito no início deste post, Beautiful Soup usa a classe <i><b>NavigableString</b></i> para retornar as strings de texto. Para fazer isso podemos usar o atributo <code>.string</code>.

Mas o método <code>get_text()</code> também retorna trechos de texto. Qual é, então, a diferença entre os dois?

Observe as funções abaixo:

{% highlight python %}
def scraper2():
    print("-----função scraper2()----")
    sectioncontent = soup.find('table', class_='no_border')
    print(sectioncontent.string)
    print("\n")

def scraper3():
    print("----função scraper3()-----")
    sectioncontent = soup.find('table', class_='no_border')
    print(sectioncontent.get_text())
{% endhighlight %}

Saída no console:
```
    -----função scraper2()----
    None


    ----função scraper3()-----




    Daily Alpha Injector



    TypeID:46375
    Published:Yes
    Volume0,01 m³
    Mass0,00 kg



    Tradeable:Yes Pilot's Services / Skill Trading
    Buildable:No 
    Inventable:No






    Group:Skill Injectors



    Category:Accessories





    MetaLevel:0
    MetaGroup:Tech I


```

A primeira função, <code>scraper2()</code>, usa a função <code>get_text()</code> para retornar o texto contido no elemento <i><b>table</b></i>, mas a sua saída no console é "None". Por sua vez, a segunda função, <code>scraper3()</code>, exibe o que realmente era desejado. Por que isso acontece?

Isso acontece porque se uma tag contiver mais de uma string, então o atributo <code>.string</code> não sabe a qual se referir. Neste caso, então, ele é definido como <i>None</i>.

o método <code>get_text()</code> retorna strings concatenadas em formato texto. Em outras palavras, <code>get_text()</code> retornará todo o texto que estiver dentro do documento/tag, em uma única string. 

Por este motivo, sempre que a extração desejada for de dados do tipo <i>plain text</i>, o método <code>get_text()</code> será mais eficaz.


## Salvando os dados

Para finalizar, salvaremos o fragmento extraído em um arquivo de texto. Mas antes disso, perceba que na saída dos dados no console, há espaçamentos excessivos.

Para resolver este problema, o Beautiful Soup nos oferece a passagem de parâmetros específicos no método <code>get_text()</code>. Usarei <code>"\n\n", strip=True</code>, para remover os espaços em branco no começo e no final de cada porção de texto, e adicionar uma dupla quebra de linha.

Feito isso, ainda temos um pequeno problema: um dos elementos veio com um link âncora que não será útil. O trecho "Pilot's Services / Skill Trading". Este trecho está dentro de uma tag <i>span</i>. Irei removê-lo acessando diretamente o atributo <code>name="1"</code> que está dentro da tag, usando o <code>.attrs</code> e usando também mais um método Beautiful Soup: o <code>clear()</code>.

{% highlight python %}
def save_scrap():
    sectioncontent = soup.find('table', class_='no_border')
    #remove o trecho “Pilot’s Services / Skill Trading”
    sp = soup.find(attrs={"name": "1"}).clear('a')
    c = sectioncontent.get_text("\n\n", strip=True)
    print(c)
    file = open('scrap.txt', 'w')
    file.write(c)     
{% endhighlight %}

A saída no console:

    Daily Alpha Injector

    TypeID:

    46375

    Published:

    Yes

    Volume

    0,01 m³

    Mass

    0,00 kg

    Tradeable:

    Yes

    Buildable:

    No

    Inventable:

    No

    Group:

    Skill Injectors

    Category:

    Accessories

    MetaLevel:

    0

    MetaGroup:

    Tech I

O arquivo, então, será salvo no caminho especificado:

![printscrap]({{"/img/printscrap.png" | absolute_url }})