# Example SignalR ASP.NET

Este é um projeto de exemplo demonstrando como implementar comunicação em tempo real usando SignalR com ASP.NET Core.

## 📋 Sobre o Projeto

SignalR é uma biblioteca que simplifica a adição de funcionalidades web em tempo real às aplicações. Este exemplo mostra como configurar e usar SignalR para criar aplicações interativas com comunicação bidirecional entre servidor e cliente.

## 🚀 Funcionalidades

- ✅ Chat em tempo real
- ✅ Notificações push
- ✅ Atualizações automáticas de dados
- ✅ Suporte a múltiplos clientes
- ✅ Grupos de usuários
- ✅ Reconexão automática

## 🛠️ Tecnologias Utilizadas

- ASP.NET Core 6.0+
- SignalR
- JavaScript/TypeScript
- HTML5/CSS3
- Entity Framework Core (opcional)

## 📦 Pré-requisitos

- .NET 6.0 SDK ou superior
- Visual Studio 2022 ou VS Code
- Navegador web moderno

## ⚙️ Instalação e Configuração

1. **Clone o repositório:**
```bash
git clone https://github.com/taviovieira89/Example_Signal_R_ASP_NET.git
```

2. **Navegue até o diretório do projeto:**
```bash
cd Example_Signal_R_ASP_NET
```

3. **Restaure as dependências:**
```bash
dotnet restore
```

4. **Execute o projeto:**
```bash
dotnet run
```

5. **Acesse a aplicação:**
   - Abra seu navegador e vá para `https://localhost:5001` ou `http://localhost:5000`

## 📁 Estrutura do Projeto

```
Example_Signal_R_ASP_NET/
├── Controllers/
│   └── HomeController.cs
├── Hubs/
│   └── ChatHub.cs
├── Models/
│   └── Message.cs
├── Views/
│   ├── Home/
│   │   └── Index.cshtml
│   └── Shared/
│       └── _Layout.cshtml
├── wwwroot/
│   ├── css/
│   ├── js/
│   │   └── chat.js
│   └── lib/
├── Program.cs
├── Startup.cs (se usando .NET 5 ou anterior)
└── appsettings.json
```

## 🔧 Configuração do SignalR

### 1. Instalação do pacote NuGet

```bash
dotnet add package Microsoft.AspNetCore.SignalR
```

### 2. Configuração no Program.cs

```csharp:Program.cs
var builder = WebApplication.CreateBuilder(args);

// Adicionar serviços
builder.Services.AddControllersWithViews();
builder.Services.AddSignalR();

var app = builder.Build();

// Configurar pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

// Mapear rotas
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Mapear Hub do SignalR
app.MapHub<ChatHub>("/chatHub");

app.Run();
```

### 3. Criação do Hub

```csharp:Hubs/ChatHub.cs
using Microsoft.AspNetCore.SignalR;

namespace Example_Signal_R_ASP_NET.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }

        public async Task JoinGroup(string groupName)
        {
            await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
            await Clients.Group(groupName).SendAsync("UserJoined", $"{Context.ConnectionId} entrou no grupo {groupName}");
        }

        public async Task LeaveGroup(string groupName)
        {
            await Groups.RemoveFromGroupAsync(Context.ConnectionId, groupName);
            await Clients.Group(groupName).SendAsync("UserLeft", $"{Context.ConnectionId} saiu do grupo {groupName}");
        }

        public override async Task OnConnectedAsync()
        {
            await Clients.All.SendAsync("UserConnected", Context.ConnectionId);
            await base.OnConnectedAsync();
        }

        public override async Task OnDisconnectedAsync(Exception exception)
        {
            await Clients.All.SendAsync("UserDisconnected", Context.ConnectionId);
            await base.OnDisconnectedAsync(exception);
        }
    }
}
```

## 💻 Exemplo de Uso no Cliente

### JavaScript

```javascript:wwwroot/js/chat.js
"use strict";

var connection = new signalR.HubConnectionBuilder()
    .withUrl("/chatHub")
    .withAutomaticReconnect()
    .build();

// Iniciar conexão
connection.start().then(function () {
    console.log("Conectado ao SignalR Hub");
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    console.error("Erro ao conectar: " + err.toString());
});

// Receber mensagens
connection.on("ReceiveMessage", function (user, message) {
    var li = document.createElement("li");
    li.textContent = `${user}: ${message}`;
    document.getElementById("messagesList").appendChild(li);
});

// Enviar mensagem
document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    
    connection.invoke("SendMessage", user, message).catch(function (err) {
        console.error("Erro ao enviar mensagem: " + err.toString());
    });
    
    document.getElementById("messageInput").value = "";
    event.preventDefault();
});

// Reconexão automática
connection.onreconnecting((error) => {
    console.log("Tentando reconectar...", error);
});

connection.onreconnected((connectionId) => {
    console.log("Reconectado com ID: " + connectionId);
});

connection.onclose((error) => {
    console.log("Conexão fechada", error);
});
```

## 🎯 Exemplos de Uso

### Chat Básico
- Envio e recebimento de mensagens em tempo real
- Notificação de usuários conectados/desconectados

### Grupos
- Criação de salas de chat
- Mensagens direcionadas para grupos específicos

### Notificações
- Alertas em tempo real
- Atualizações de status

## 🔒 Segurança

Para ambientes de produção, considere implementar:

- Autenticação e autorização
- Validação de entrada
- Rate limiting
- CORS apropriado
- HTTPS obrigatório

```csharp:Program.cs
// Exemplo de configuração de CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("CorsPolicy", builder =>
    {
        builder
            .WithOrigins("https://localhost:3000")
            .AllowAnyMethod()
            .AllowAnyHeader()
            .AllowCredentials();
    });
});

// Usar CORS
app.UseCors("CorsPolicy");
```

## 🧪 Testes

Para executar os testes:

```bash
dotnet test
```

## 📚 Recursos Adicionais

- [Documentação oficial do SignalR](https://docs.microsoft.com/pt-br/aspnet/core/signalr/)
- [Tutorial SignalR](https://docs.microsoft.com/pt-br/aspnet/core/tutorials/signalr)
- [SignalR JavaScript Client](https://docs.microsoft.com/pt-br/aspnet/core/signalr/javascript-client)

## 🤝 Contribuição

1. Faça um fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

## 👨‍💻 Autor

linkedin: https://www.linkedin.com/in/t%C3%A1vio-vieira-ara%C3%BAjo-70a17979/

Link do Projeto: [https://github.com/taviovieira89/Example_Signal_R_ASP_NET](https://github.com/taviovieira89/Example_Signal_R_ASP_NET)

## 🆘 Suporte

Se você encontrar algum problema ou tiver dúvidas:

1. Verifique as [Issues](https://github.com/taviovieira89/Example_Signal_R_ASP_NET/issues) existentes
2. Crie uma nova issue se necessário
3. Entre em contato através do email: taviovieira89@exemplo.com

---

⭐ Se este projeto te ajudou, considere dar uma estrela no repositório!