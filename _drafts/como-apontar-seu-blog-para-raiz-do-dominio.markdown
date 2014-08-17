---
layout: post
title:  "Como apontar seu blog para raiz do domínio"
date:   2014-08-16 22:50:02
categories: infrastructure dns tips
---

Já há algum tempo, algumas pessoas têm me perguntado como configurar o DNS para que os visitantes de seus sites ou blogs, pudessem acessá-los digitando somente o nome do domínio. Ou seja, dando o exemplo este blog, eles me perguntam como fazer para que uma pessoa possa acessa-lo digitando [www.cefig.net](http://www.cefig.net) ou somente [cefig.net](http://cefig.net).

Esse tipo de comportamento pode ser facilmente conseguido via configuração dos servidores DNS. Pretendo explicar como fazer para alguns dos cenários mais comuns. Mas primeiro preciso introduzir alguns termos de conviguração de servidores DNS.

## Básico sobre DNS

Basicamente, [DNS ou Domain Name System](http://en.wikipedia.org/wiki/Domain_Name_System) é um serviço que recebe como requisição um nome e responde com o endereço IP associado aquele nome. O nome é uma composição hierárquica de domínios e subdominios. Onde para cada domínio, existe dois ou mais servidores DNS com autoridade por responder por todos os endereços deste domínio. Estes servidores recebem a alcunha de [Name Server](http://en.wikipedia.org/wiki/Name_server). 

Hoje em dia, ao contratarmos nossos domínios em serviços como [GoDaddy](http://godaddy.com) e [Registro.br](http://registro.br) já nos é associado automaticamente para nosso novo domínio, os servidor do tipo NS necessários para nosso domínio funcionar sem problemas, onde nós só precisamos cadastrar os endereços dos nossos subdomínios e serviços, como email, site, endereço de servidor. E esses serviços são cadastrados ao incluirmos registros dos tipos:

### A:
Registros de tipo `A`, são usados para atribuir um alias a um endereço IP, para que possamos referenciar este IP usando nosso nome de dominio. 
Por exemplo, se eu tivesse no meu dns a configuração: 
        
        api     A    122.55.22.23

Existiria no meu domínio o endereço api.cefig.net  que me direcionaria para o IP 122.55.22.23

*Importante: Um endereço do tipo `A` aponta somente para endereços IP!*

### CNAME
Registros do tipo CNAME servem para atribuir um "apelido" a algum outro alias já mapeado. Numa configuração de CNAME, você não aponta para um IP, e sim para algum nome que exista. Ou seja, com um CNAME você ponde criar apelidos para endereços A existentes, ou mesmo criar um apelido para um endereço que sequer seja do seu domínio. Exemplo:

        www     CNAME   cefigueiredo.github.io
        api2    CNAME   api

No primeiro exemplo, eu listei o endereço WWW do meu blog... [www.cefig.net](http://www.cefig.net) que está redirecionando para o verdadeiro local do meu blog, que é o endereço [cefigueiredo.github.io](http://cefigueiredo.github.io)
Já no segundo, montei uma apelido fictício, cujo resultado seria o nome `api2.cefig.net` que assim como o nome `api.cefig.net` resultaria no endereço IP `122.55.22.23`.


Existem outros tipos de registros DNS, como o MX, o próprio NS, TXT... uma lista completa pode ser encontrada [aqui](http://en.wikipedia.org/wiki/List_of_DNS_record_types). Porém para este post, somente os endereços A e CNAME são necessários.

## Mãos a obra