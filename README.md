# SignalRChat
<h2>This tutorial teaches the basics of building a real-time app using SignalR.</h2>
You learn how to:

-    Create a web project.
-    Add the SignalR client library.
-    Create a SignalR hub.
-    Configure the project to use SignalR.
-    Add code that sends messages from any client to all connected clients.

At the end, you'll have a working chat app.  

<h3>Prerequisites</h3>

Visual Studio 2019 with the ASP.NET and web development workload .NET Core 3.0 SDK or later.

<h3>Create a web app project</h3>

-    From the menu, select File > New Project.
-    In the Create a new project dialog, select ASP.NET Core Web Application, and then select Next.
-    In the Configure your new project dialog, name the project SignalRChat, and then select Create.
-    In the Create a new ASP.NET Core web Application dialog, select .NET Core and ASP.NET Core 3.0.
-    Select Web Application to create a project that uses Razor Pages, and then select Create.

<h3>Add the SignalR client library</h3>

The SignalR server library is included in the ASP.NET Core 3.0 shared framework. The JavaScript client library isn't automatically included in the project. For this tutorial, you use Library Manager (LibMan) to get the client library from unpkg. unpkg is a content delivery network (CDN) that can deliver anything found in npm, the Node.js package manager.

-    In Solution Explorer, right-click the project, and select Add > Client-Side Library.
-    In the Add Client-Side Library dialog, for Provider select unpkg.
-    For Library, enter @microsoft/signalr@latest.
-    Select Choose specific files, expand the dist/browser folder, and select signalr.js and signalr.min.js.
-    Set Target Location to wwwroot/js/signalr/, and select Install.
-    Add Client-Side Library dialog - select library

LibMan creates a wwwroot/js/signalr folder and copies the selected files to it.

<h3>Create a SignalR hub</h3>

A hub is a class that serves as a high-level pipeline that handles client-server communication.

-    In the SignalRChat project folder, create a Hubs folder.
-    In the Hubs folder, create a ChatHub.cs file with the following code:
```
using Microsoft.AspNetCore.SignalR;
using System.Threading.Tasks;

namespace SignalRChat.Hubs
{
    public class ChatHub : Hub
    {
        public async Task SendMessage(string user, string message)
        {
            await Clients.All.SendAsync("ReceiveMessage", user, message);
        }
    }
}
```

The ChatHub class inherits from the SignalR Hub class. The Hub class manages connections, groups, and messaging.
The SendMessage method can be called by a connected client to send a message to all clients. JavaScript client code that calls the method is shown later in the tutorial. SignalR code is asynchronous to provide maximum scalability.

<h3>Configure SignalR</h3>

The SignalR server must be configured to pass SignalR requests to SignalR.
Replace the following content to the Startup.cs file.

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.HttpsPolicy;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using SignalRChat.Hubs;

namespace SignalRChat
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddRazorPages();
            services.AddSignalR();
        }

        // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            app.UseHttpsRedirection();
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapRazorPages();
                endpoints.MapHub<ChatHub>("/chathub");
            });
        }
    }
}
```

These changes add SignalR to the ASP.NET Core dependency injection and routing systems.

<h3>Add SignalR client code</h3>

Replace the content in Pages\Index.cshtml with the following code:
```
@page
    <div class="container">
        <div class="row">&nbsp;</div>
        <div class="row">
            <div class="col-2">User</div>
            <div class="col-4"><input type="text" id="userInput" /></div>
        </div>
        <div class="row">
            <div class="col-2">Message</div>
            <div class="col-4"><input type="text" id="messageInput" /></div>
        </div>
        <div class="row">&nbsp;</div>
        <div class="row">
            <div class="col-6">
                <input type="button" id="sendButton" value="Send Message" />
            </div>
        </div>
    </div>
    <div class="row">
        <div class="col-12">
            <hr />
        </div>
    </div>
    <div class="row">
        <div class="col-6">
            <ul id="messagesList"></ul>
        </div>
    </div>
<script src="~/js/signalr/dist/browser/signalr.js"></script>
<script src="~/js/chat.js"></script>
```
The preceding code:

-    Creates text boxes for name and message text, and a submit button.
-    Creates a list with id="messagesList" for displaying messages that are received from the SignalR hub.
-    Includes script references to SignalR and the chat.js application code that you create in the next step.

In the wwwroot/js folder, create a chat.js file with the following code:
```
"use strict";

var connection = new signalR.HubConnectionBuilder().withUrl("/chatHub").build();

//Disable send button until connection is established
document.getElementById("sendButton").disabled = true;

connection.on("ReceiveMessage", function (user, message) {
    var msg = message.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
    var encodedMsg = user + " says " + msg;
    var li = document.createElement("li");
    li.textContent = encodedMsg;
    document.getElementById("messagesList").appendChild(li);
});

connection.start().then(function () {
    document.getElementById("sendButton").disabled = false;
}).catch(function (err) {
    return console.error(err.toString());
});

document.getElementById("sendButton").addEventListener("click", function (event) {
    var user = document.getElementById("userInput").value;
    var message = document.getElementById("messageInput").value;
    connection.invoke("SendMessage", user, message).catch(function (err) {
        return console.error(err.toString());
    });
    event.preventDefault();
});
```

The preceding code:

-    Creates and starts a connection.
-    Adds to the submit button a handler that sends messages to the hub.
-    Adds to the connection object a handler that receives messages from the hub and adds them to the list.

<h3>Run the app</h3>

-    Press CTRL+F5 to run the app without debugging.
-    Copy the URL from the address bar, open another browser instance or tab, and paste the URL in the address bar.
-    Choose either browser, enter a name and message, and select the Send Message button.
-    The name and message are displayed on both pages instantly.

<h3>Tip</h3>

If the app doesn't work, open your browser developer tools (F12) and go to the console. You might see errors related to your HTML and JavaScript code. For example, suppose you put signalr.js in a different folder than directed. In that case the reference to that file won't work and you'll see a 404 error in the console. signalr.js not found error

If you get the error ERR_SPDY_INADEQUATE_TRANSPORT_SECURITY in Chrome, run these commands to update your development certificate:
```
dotnet dev-certs https --clean
dotnet dev-certs https --trust
```
