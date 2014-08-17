---
layout: post
title:  "Como apontar seu blog para raiz do domínio"
date:   2014-08-17 17:23:32
categories: infrastructure dns tips
---

Já há algum tempo, algumas pessoas têm me perguntado como configurar o DNS para que os visitantes de seus sites ou blogs, pudessem acessá-los digitando somente o nome do domínio. Ou seja, dando o exemplo deste blog, eles me perguntam como fazer para que uma pessoa possa acessa-lo digitando somente [cefig.net](http://cefig.net).

Esse tipo de comportamento pode ser facilmente conseguido via configuração dos servidores DNS. Pretendo explicar como fazer para alguns dos cenários mais comuns. Mas primeiro preciso introduzir alguns termos de configuração de servidores DNS, e em seguida mostrar como mapear um endereço para ser alcançado quando alguem tentar acessar a raiz do domínio.

## Básico sobre DNS

Basicamente, [DNS ou Domain Name System](http://en.wikipedia.org/wiki/Domain_Name_System) é um serviço que recebe como requisição um nome e responde com o endereço IP associado a aquele nome. O nome é uma composição hierárquica de domínios e subdominios. Onde para cada domínio, existe dois ou mais servidores DNS com autoridade por responder por todos os endereços deste domínio, ou redirecionar para outro servidor responsável por fazê-lo. Estes servidores recebem a alcunha de [Name Server](http://en.wikipedia.org/wiki/Name_server). 

Hoje em dia, ao contratarmos nossos domínios em serviços como [GoDaddy](http://godaddy.com) e [Registro.br](http://registro.br) já nos é associado automaticamente para nosso novo domínio, os servidor do tipo NS necessários para nosso domínio funcionar sem problemas, onde nós só precisamos cadastrar os endereços dos nossos subdomínios e serviços, como email, site, endereço de servidor. E esses serviços são cadastrados ao incluirmos registros dos tipos:

### A (Host):
Registros de tipo `A`, são usados para atribuir um nome de "host" a um endereço IP, para que possamos referenciar este IP usando nosso nome de dominio. 
Por exemplo, se eu tivesse no meu dns a configuração: 
        
        # nome    tipo    aponta para
          api     A       122.55.22.23

Existiria no meu domínio o endereço api.cefig.net  que me direcionaria para o IP 122.55.22.23

*Importante: Um endereço do tipo `A` aponta somente para endereços IP!*

### CNAME (Alias):
Registros do tipo CNAME servem para atribuir um "alias" a algum host já mapeado. Numa configuração de CNAME, você não aponta para um IP, e sim para algum nome que exista. Ou seja, com um CNAME você pode criar apelidos para hosts A existentes, ou mesmo criar um apelido para um endereço que sequer seja do seu domínio. Exemplo:

        # nome    tipo    aponta para
          www     CNAME   cefigueiredo.github.io
          api2    CNAME   api

No primeiro exemplo, eu listei o endereço `WWW` do meu blog... [www.cefig.net](http://www.cefig.net) que está redirecionando para o verdadeiro local do meu blog, que é o endereço [cefigueiredo.github.io](http://cefigueiredo.github.io)
Já no segundo, montei uma apelido para um host do meu domínio, cujo resultado seria o nome `api2.cefig.net` que assim como o nome `api.cefig.net` resultaria no endereço IP `122.55.22.23`.


Existem outros tipos de registros DNS, como o MX, o próprio NS, TXT... uma lista completa pode ser encontrada [aqui](http://en.wikipedia.org/wiki/List_of_DNS_record_types). Porém para este post, somente os endereços `A` e `CNAME` são necessários.

Antes de botar a mão na massa de fato, preciso falar de um outro termo quando tratamos de DNS, que é o `root domain`, que nada mais é que o endereço raiz do seu domínio. No meu caso, meu root domain é `cefig.net`. Tudo que eu configurar no meu DNS tera como sufixo o root domain, por isso quando configuro um Alias `api` ou CNAME `api2` o resultado é respectivamente `api.cefig.net` e `api2.cefig.net`.

## Mãos a obra

Como disse, para permitir que quem acesse o seu root domain, seja encaminhado para o endereço de seu blog. Só é preciso a configuração do seu serviço DNS. Como mostrarei a seguir. 

Se você tem um blog, provavelmente você contratou seu domínio em algum serviço como o [Registro.BR][registrobr], [GoDaddy][godaddy], ou algo do gênero. O importante é que ao contratar seu domínio, mesmo não sabendo, você recebeu acesso a alguma plataforma de gestão do seu domínio, e esta plataforma vai ser sua interface para configuração do serviço DNS, onde você seguirá os passos descritos neste post. 

***Disclaimer: Não tenho acesso a muitas plataformas, tampouco é o objetivo do post mostrar como ter acesso a elas, visto que esta informação pode ser obtida nas páginas de suporte do seu fornecedor de serviços de dominio, procurando por `Confiruração DNS` ou `DNS Manager`. Portanto me concentrarei somente na configuração do DNS, que é relativamente comum a qualquer plataforma, podendo somente ter alguns termos diferentes, porém com mesma essência.*** 

Configurando somente o DNS, o que precisamos fazer é associar um endereço ao host `@`.
O `@`, numa configuração DNS é só um wildcard que referencia o próprio root domain. Logo, quando associamos algum endereço ao nome `@` estamos na verdade atribuindo um endereço para o root domain. E só isso já é o que precisamos para fazer com que o root domain direcione para o mesmo endereço associado ao seu host `blog` ou `www`.

Existem 2 modos de se atribuir este nome. O modo mais simples, e o que tem 100% de certeza de ser possível de configurar em qualquer domínio. É criar um registro do tipo `A` com o nome `@` assim:

        # nome   tipo     aponta para
          @      A        122.22.22.22


O outro modo, é bem semelhante, o que difere é que usaremos registros CNAME.

Como já disse anteriormente, o host `A` só aceita apontar para endereços IP. Então se você não tiver um endereço de IP fixo para o seu blog , por exemplo, se seu blog/site estiver no [Heroku](http://heroku.com) ou [Wordpress](http://wordpress.com)), então você precisará criar um alias `CNAME` que aponte para o endereço do seu blog/site.

Em tese, hoje em dia qualquer servidor DNS permite associar um registro CNAME ao wildcard `@`, porém, nem todas as interfaces WEB de gestão de DNS permitem que os usuários façam esse tipo de configuração. Pois como um registro CNAME direciona para qualquer endereço, existiria o risco de uma pessoa leiga configurar incorretamente o CNAME, e permitir que o serviço DNS entre em um loop que acabaria derrubando o serviço de DNS por inconsistência.
As únicas plataformas que tenho acesso são [GoDaddy][godaddy] e [Registro.BR][registrobr], e nenhuma dessas duas me permite associar o wildcard `@` a um registro `CNAME`. Mas em plataformas como a [DNSimple](http://dnsimple.com) esse tipo de configuração é possível. E ficaria assim:

      # nome    tipo        aponta para
        @       CNAME       www
        @       CNAME       my_fancy_blog.herokuapp.com

Assim como mencionei antes, o `CNAME` pode apontar para um nome que já exista no seu domínio (mesmo se for para outro `CNAME`) ou para um nome de outro domínio.

Configurando somente DNS, estes são os dois modos possíveis de permitir que ao acessar seu root domain, seja possível chegar ao endereço de onde seu blog está hospedado.

## O caso Github Pages

Como eu disse anteriormente, para criarmos registros do tipo `A`, é necessário apontar para endereços IP, e nem todos os servidores de DNS, permitem configurar um registro CNAME usando o wildcard do root domain. Sabendo disso, o [Github Pages][GithubPages] criou um serviço para que seja possível criar registros do tipo `A` para o wildcard `@ `.

Na verdade, o serviço consiste em fornecer 2 ips fixos, para onde são encaminhadas as requisições de quem tenta acessar o seu blog.

Para habilitar é preciso seguir os seguintes passos:

### Criar os registros `A` no DNS.

        # nome    tipo    aponta para
          @       A       192.30.252.153
          @       A       192.30.252.154

### Criar um arquivo `CNAME`

Para identificar o seu blog, dentre todos os outros, o github precisa que você commite junto ao seu blog, um arquivo chamado `CNAME`.
Neste arquivo você só precisa colocar o nome do seu domínio. Isso já será suficiente para o github conseguir identificar para qual blog redirecionar quando alquem tentar acessar algum desses endereços IP vindos do seu domínio.

Sem configurar o root domain para ser redirecionado para o seu blog, é possível criar qualquer registro `CNAME` (no seu DNS) apontando para `username.github.io`. Porém, se você fizer a configuração do seu root domain nos passos anteriores, o único registro `CNAME` que o Github irá permitir alcançar o seu blog, será o `www`, como mostrado [aqui](https://help.github.com/articles/tips-for-configuring-an-a-record-with-your-dns-provider)!

Enfim, com essas informações, creio já ser possível configurar seu root domain perfeitamente. Então, se você seguir esses procedimentos (e leu até aqui), me deixe saber se funcionou ou se teve alguma dificuldade, para melhorar!



[GithubPages]: http://pages.github.com
[godaddy]: http://godaddy.com
[registrobr]: http://registro.br