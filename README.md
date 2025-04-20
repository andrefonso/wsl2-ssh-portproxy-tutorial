# wsl2-ssh-portproxy-tutorial
Este repositório documenta o passo a passo para permitir conexões SSH externas ao Ubuntu rodando no WSL2, a partir de outros dispositivos na mesma rede local.

<img src="/imagens/wsl2ssh.png">

# Acessando o Ubuntu no WSL2 via SSH de outros dispositivos da rede local

Este repositório documenta o passo a passo para permitir conexões SSH externas ao Ubuntu rodando no WSL2, a partir de outros dispositivos na mesma rede local.

## 🧠 Por que isso é necessário?

Por padrão, o WSL2 roda em uma **interface de rede virtual isolada**, com um IP próprio (ex: `172.25.x.x`) que **não é visível diretamente na rede local**. Isso impede que outros computadores acessem diretamente os serviços do Ubuntu no WSL2, como o SSH.

Mesmo que o SSH esteja rodando no Ubuntu do WSL2, tentar se conectar a partir de outro computador da rede resulta em erro como:

ssh: connect to host 172.25.170.81 port 22: Connection timed out

Esse IP só existe **dentro do Windows**. Para contornar esse isolamento, usamos uma funcionalidade do Windows chamada `portproxy`.

---

## 🚀 Solução: redirecionar a porta 22 do Windows para o Ubuntu no WSL2

A ideia é fazer o Windows "escutar" em sua interface de rede local (ex: `192.168.0.101`) e **repassar conexões para o IP do WSL2**, na mesma porta.

### Exemplo:

Windows IP (host): 192.168.0.101 Ubuntu WSL2 IP: 172.25.170.81 Porta SSH: 22


## Passos a seguir:

1) Descubra o IP do Windows na rede local com:

 ```powershell
ipconfig
```

Anote o IP IPv4 da sua conexão principal. Exemplo: 192.168.0.101 (caso o seu IP seja outro, basta substituir por ele nos comandos a seguir)

2) Execute o seguinte comando como administrador no PowerShell:

```powershell
netsh interface portproxy add v4tov4 listenport=22 listenaddress=192.168.0.101 connectport=22 connectaddress=$($(wsl hostname -I).Trim())
```
Este comando cria uma regra de redirecionamento de IP. Caso o IP digitado tenha sido incorreto, a remoção da regra pode ser efetuada com o seguinte comando:

```powershell
netsh interface portproxy delete v4tov4 listenport=22 listenaddress=192.168.0.101
```

3) Para confirmar que funcionou:

```powershell
netsh interface portproxy show v4tov4
```

Saída esperada:

```
Listen on ipv4:                  Connect to ipv4:

Address        Port              Address         Port
-------------  -----             -------------   -----
192.168.0.101  22                172.25.170.81   22
```

4) Inicie (ou reinicie) o SSH no WSL2:

```sh
sudo service ssh start
```

5) Acesse de outro computador
   A partir de outro PC na mesma rede:

```sh
ssh usuario@192.168.0.101
```


