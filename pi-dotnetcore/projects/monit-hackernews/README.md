## Hackernews Monitoring Service

Scenario:
- Deploying an ASP.NET Core service + SignalR hub on Raspberry Pi to monitor Hackernews
- When latest news is updated, the Raspberry Pi will update connected clients using SignalR

Requirements: dotnet core 3.1 (https://dotnet.microsoft.com/download/dotnet-core/3.1)

### Server
#### On Development Machine
From a command prompt:

```
cd monit-hackernews
dotnet add package Microsoft.AspNetCore.SignalR.Client
dotnet watch run
```

Navigate to https://localhost:5001

Package into a self-contained deployment (https://docs.microsoft.com/en-us/dotnet/core/deploying/deploy-with-cli)
```
dotnet publish --runtime linux-arm --self-contained true --configuration Release
```
Copy the contents of bin/Release/netcoreapp3.1/linux-arm/publish to the Raspberry Pi.

#### On Raspberry Pi

1. Kestrel (the cross-platform ASP.NET core web server) requires an SSL certificate. Generate the certificate:
```
cd docker

openssl req \
 -newkey rsa:2048 \
 -nodes \
 -keyout pi.key \
 -out pi.csr

openssl x509 \
 -signkey pi.key \
 -in pi.csr \
 -req \
 -days 3650 \
 -out pi.crt

openssl pkcs12 \
 -inkey pi.key \
 -in pi.crt \
 -export \
 -out pi.pfx
```

2. Update `docker/.env` to set SSL_PASS to the password you used for generating the cert.

3. Run docker compose, which will launch the server with the SSL cert path and password specified in the .env.
```
docker-compose up
```

Navigate to https://ipaddress_of_pi:18081

![example](example.png)

### Client

From a command prompt:

```
cd monit-hackernews/client
dotnet run [ip_address_of_server(default: localhost)] [port(default: 5001)]
```

![client](client.png)

### References
- https://codedbeard.com/iot-with-blazor-on-raspberry-pi-part-3/
- https://github.com/HackerNews/API
- https://docs.microsoft.com/en-us/aspnet/core/tutorials/signalr?view=aspnetcore-3.1
- https://mohitgoyal.co/2018/09/25/use-ssl-certificates-for-dotnet-core-application-in-docker-containers/
- https://medium.com/@sketchmycloud/how-to-deploy-asp-net-core-2-1-applications-on-ubuntu-474062f8db73

