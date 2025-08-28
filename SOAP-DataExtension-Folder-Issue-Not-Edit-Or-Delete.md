# 🧩 Problema ao Criar Pasta de Data Extension via SOAP API no Salesforce Marketing Cloud

## 📌 Sumário
- [Descrição do Problema](#descrição-do-problema)
- [Causa do Problema](#causa-do-problema)
- [Solução](#solução)
  - [1. Recuperar o `ObjectID` da Pasta](#1-recuperar-o-objectid-da-pasta)
  - [2. Atualizar as Propriedades da Pasta](#2-atualizar-as-propriedades-da-pasta)
- [SOAP Requests](#soap-requests)
  - [Retrieve da Pasta Principal de Data Extensions](#retrieve-da-pasta-principal-de-data-extensions)
  - [Retrieve do `ObjectID` da Pasta Criada](#retrieve-do-objectid-da-pasta-criada)
  - [Update para Habilitar Edição e Exclusão](#update-para-habilitar-edição-e-exclusão)
- [Resumo da Solução](#resumo-da-solução)

---

## 📌 Descrição do Problema

Durante o desenvolvimento com **Salesforce Marketing Cloud**, criei uma pasta de **Data Extension** via **SOAP API**.  
No entanto, após a criação, **não era possível excluir nem editar** essa pasta.

---

## 🛑 Causa do Problema

Quando criamos uma pasta de **Data Extension** via SOAP API, algumas propriedades **não são habilitadas por padrão**, o que bloqueia qualquer alteração futura.

As propriedades que estavam desabilitadas eram:

- `AllowChildren` → Define se a pasta pode ter subpastas.
- `IsActive` → Define se a pasta está ativa.
- `IsEditable` → Define se a pasta pode ser editada ou excluída.

---

## ✅ Solução

Para resolver, foram necessários **dois passos**:

---

### **1. Recuperar o `ObjectID` da Pasta**

Primeiro, executei um **Retrieve** para buscar o **ObjectID** da pasta criada:

```xml
<s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:a="http://schemas.xmlsoap.org/ws/2004/08/addressing">
    <s:Header>
        <a:Action s:mustUnderstand="1">Retrieve</a:Action>
        <a:To s:mustUnderstand="1">yourSoapInstanceUrl/Service.asmx</a:To>
        <fueloauth xmlns="http://exacttarget.com">yourAccessToken</fueloauth>
    </s:Header>
    <s:Body>
        <RetrieveRequestMsg xmlns="http://exacttarget.com/wsdl/partnerAPI">
            <RetrieveRequest>
                <ObjectType>DataFolder</ObjectType>
                <Properties>ObjectID</Properties>
                <Properties>ID</Properties>
                <Filter xsi:type="SimpleFilterPart" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
                    <Property>Name</Property>
                    <SimpleOperator>equals</SimpleOperator>
                    <Value>DE_CREATOR</Value>
                </Filter>
            </RetrieveRequest>
        </RetrieveRequestMsg>
    </s:Body>
</s:Envelope>
```

### **2. Alterar propriedades da Pasta**

Após obter o **ObjectID**, executei um **Update** para habilitar as propriedades:

```xml
<s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:a="http://schemas.xmlsoap.org/ws/2004/08/addressing">
    <s:Header>
        <a:Action s:mustUnderstand="1">Update</a:Action>
        <a:To s:mustUnderstand="1">yourSoapInstanceUrl/Service.asmx</a:To>
        <fueloauth xmlns="http://exacttarget.com">yourAccessToken</fueloauth>
    </s:Header>
    <s:Body xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
        <UpdateRequest xmlns="http://exacttarget.com/wsdl/partnerAPI">
            <Objects xsi:type="DataFolder">
                <ObjectID>yourFolderObjectID</ObjectID>
                <AllowChildren>true</AllowChildren>
                <IsActive>true</IsActive>
                <IsEditable>true</IsEditable>
            </Objects>
        </UpdateRequest>
    </s:Body>
</s:Envelope>
```

## 📎 Dica
Sempre que criar pastas via SOAP, defina as propriedades corretas já na criação para evitar esse problema no futuro.


