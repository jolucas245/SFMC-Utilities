# 📂 Criar Pasta e Data Extension Dentro de "Subscribers > Data Extensions" via SOAP API

## 📌 Descrição do Problema

Ao criar **pasta** e **Data Extension** via **SOAP API** no **Salesforce Marketing Cloud**, é necessário informar o `CategoryID` (ID da pasta onde a Data Extension será criada).  
O problema é que o `CategoryID` da pasta **Data Extensions** varia **de cliente para cliente**.

---

## 🛑 Causa do Problema

Quando queremos criar uma **Data Extension** diretamente via SOAP, precisamos informar o **CategoryID** da pasta onde ela será criada.  
Entretanto, como o **ID da pasta raiz "Data Extensions"** muda em cada **Business Unit** e em cada **cliente**, **não é possível usar um ID fixo**.

---

## ✅ Solução

Para contornar essa limitação, a solução foi **dinâmica** e composta de três etapas:

---

### **1. Recuperar o ID da Pasta Raiz "Data Extensions"**

A pasta raiz **Data Extensions** existe em **todas** as instâncias do Salesforce Marketing Cloud.  
O primeiro passo é recuperar o **ID** dela com um **Retrieve**:

```xml
<s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:a="http://schemas.xmlsoap.org/ws/2004/08/addressing">
    <s:Header>
        <a:Action s:mustUnderstand="1">Retrieve</a:Action>
        <a:To s:mustUnderstand="1">${tokenResponse.soapInstanceUrl}Service.asmx</a:To>
        <fueloauth xmlns="http://exacttarget.com">${tokenResponse.accessToken}</fueloauth>
    </s:Header>
    <s:Body xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
        <RetrieveRequestMsg xmlns="http://exacttarget.com/wsdl/partnerAPI">
            <RetrieveRequest>
                <ObjectType>DataFolder</ObjectType>
                <Properties>ID</Properties>
                <Properties>Name</Properties>
                <Properties>ParentFolder.ID</Properties>
                <Filter xsi:type="ComplexFilterPart">
                    <LeftOperand xsi:type="SimpleFilterPart">
                        <Property>Name</Property>
                        <SimpleOperator>equals</SimpleOperator>
                        <Value>Data Extensions</Value>
                    </LeftOperand>
                    <LogicalOperator>AND</LogicalOperator>
                    <RightOperand xsi:type="SimpleFilterPart">
                        <Property>ContentType</Property>
                        <SimpleOperator>equals</SimpleOperator>
                        <Value>dataextension</Value>
                    </RightOperand>
                </Filter>
            </RetrieveRequest>
        </RetrieveRequestMsg>
    </s:Body>
</s:Envelope>
```
**Resposta esperada:** O campo ID retornado será o CategoryID da pasta Data Extensions.

### **2. Recuperar o ID da Pasta Raiz "Data Extensions"**

Com o **ID da pasta raiz**, podemos criar uma subpasta dentro de **Data Extensions**:

```xml
<s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:a="http://schemas.xmlsoap.org/ws/2004/08/addressing">
    <s:Header>
        <a:Action s:mustUnderstand="1">Create</a:Action>
        <a:To s:mustUnderstand="1">${tokenResponse.soapInstanceUrl}Service.asmx</a:To>
        <fueloauth xmlns="http://exacttarget.com">${tokenResponse.accessToken}</fueloauth>
    </s:Header>
    <s:Body xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
        <CreateRequest xmlns="http://exacttarget.com/wsdl/partnerAPI">
            <Objects xsi:type="DataFolder">
                <Name>${name}</Name>
                <Description>${description}</Description>
                <ContentType>dataextension</ContentType>
                <ParentFolder>
                    <ID>${parentId}</ID>
                </ParentFolder>
            </Objects>
        </CreateRequest>
    </s:Body>
</s:Envelope>
```

**Dica**:
- ${name} → Nome da nova pasta.
- ${description} → Descrição da pasta.
- ${parentId} → ID obtido no passo 1.

### **3. Criar a Data Extension Dentro da Pasta Criada**

Após criar a pasta, precisamos usar o **ID dela** (`folderId`) como **CategoryID** na criação da Data Extension:

```xml
<s:Envelope xmlns:s="http://www.w3.org/2003/05/soap-envelope" xmlns:a="http://schemas.xmlsoap.org/ws/2004/08/addressing">
    <s:Header>
        <a:Action s:mustUnderstand="1">Create</a:Action>
        <a:To s:mustUnderstand="1">${tokenResponse.soapInstanceUrl}Service.asmx</a:To>
        <fueloauth xmlns="http://exacttarget.com">${tokenResponse.accessToken}</fueloauth>
    </s:Header>
    <s:Body xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">
        <CreateRequest xmlns="http://exacttarget.com/wsdl/partnerAPI">
            <Objects xsi:type="DataExtension">
                <CategoryID>${folderId}</CategoryID>
                <Name>${name}</Name>
                <CustomerKey>${customerKey}</CustomerKey>
                <Description>${description}</Description>
                <IsSendable>false</IsSendable>
                <Fields>
                    ${fieldsXml}
                </Fields>
            </Objects>
        </CreateRequest>
    </s:Body>
</s:Envelope>
```

**Dica**:
- ${folderId} → ID da pasta criada.
- ${customerKey} → Chave única da Data Extension.
- ${fieldsXml} → Estrutura XML com os campos da Data Extension.

| Finalidade                                   | Método   | Objeto        | Campos Importantes        |
| -------------------------------------------- | -------- | ------------- | ------------------------- |
| Buscar ID da pasta **Data Extensions**       | Retrieve | DataFolder    | `ID`, `Name`              |
| Criar **subpasta** dentro de Data Extensions | Create   | DataFolder    | `Name`, `ParentFolder.ID` |
| Criar **Data Extension** na pasta criada     | Create   | DataExtension | `CategoryID`, `Fields`    |

## 🔗 Próximo Passo

Se você também precisa atualizar propriedades da pasta (por exemplo, `AllowChildren`, `IsEditable`, `IsActive`), veja a documentação [deste outro caso](./SOAP-DataExtension-Folder-Issue.md).
