# Create .NetCore WebApp 

  https://www.yogihosting.com/category/aspnet-core/
  https://www.tutorialsteacher.com/core/net-core-command-line-interface

  PS C:\K8S-Demo> dotnet new webapp  -n HelloWorld -f net6.0      
  The template "ASP.NET Core Web App" was created successfully.
  This template contains technologies from parties other than Microsoft, see https://aka.ms/aspnetcore/6.0-third-party-notices for details.

  Processing post-creation actions...
  Restoring C:\K8S-Demo\HelloWorld\HelloWorld.csproj:
    Determining projects to restore...
    Restored C:\K8S-Demo\HelloWorld\HelloWorld.csproj (in 142 ms).
  Restore succeeded.

  PS C:\K8S-Demo> dotnet build .\HelloWorld\HelloWorld.csproj         
  MSBuild version 17.5.0-preview-23061-01+040e2a90e for .NET
    Determining projects to restore...
    All projects are up-to-date for restore.
    HelloWorld -> C:\K8S-Demo\HelloWorld\bin\Debug\net6.0\HelloWorld.dll

  Build succeeded.
    0 Warning(s)
    0 Error(s)


  PS C:\K8S-Demo> dotnet run --project .\HelloWorld\HelloWorld.csproj
  Building...
  info: Microsoft.Hosting.Lifetime[14]
        Now listening on: https://localhost:7064
  info: Microsoft.Hosting.Lifetime[14]
        Now listening on: http://localhost:5181
  info: Microsoft.Hosting.Lifetime[0]
        Application started. Press Ctrl+C to shut down.
  info: Microsoft.Hosting.Lifetime[0]
        Hosting environment: Development
  info: Microsoft.Hosting.Lifetime[0]
        Content root path: C:\K8S-Demo\HelloWorld\

# Add Dockerfile to Workspace

  - Install C# extention (C# for Visual Studio Code (powered by OmniSharp))
  - Command Palette (Ctrl+Shift+P)
      View-> Command Palette 
  - Add dockeer file to namespace

# Add .gitignore file 
  Add the Extention .gitignore Generator
  In VS code Command Palette Add .gitignore

# Docker Build
  PS C:\K8S-Demo> docker build -t helloworld .\HelloWorld 
  Sending build context to Docker daemon  9.817MB
  Step 1/16 : FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
  ---> d18ea9795cde
  Step 2/16 : WORKDIR /app
  ---> Using cache
  ---> 52ada268b341
  Step 3/16 : EXPOSE 80
  ---> Using cache
  ---> abf2dca9ebac
  Step 4/16 : FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
  ---> 7b6a689812d3
  Step 5/16 : WORKDIR /src
  ---> Using cache
  ---> 6cdbf7f95044
  Step 6/16 : COPY ["HelloWorld.csproj", "."]
  ---> 19f3bd859cf5
  Step 7/16 : RUN dotnet restore "./HelloWorld.csproj"
  ---> Running in c453b3d5e994
    Determining projects to restore...
    Restored /src/HelloWorld.csproj (in 134 ms).
  Removing intermediate container c453b3d5e994
  ---> 9fe3013e0697
  Step 8/16 : COPY . .
  ---> 9f32611d9266
  Step 9/16 : WORKDIR "/src/."
  ---> Running in 5066f4ad5ffb
  Removing intermediate container 5066f4ad5ffb
  ---> 1e43a5d175f6
  Step 10/16 : RUN dotnet build "HelloWorld.csproj" -c Release -o /app/build
  ---> Running in 30866e9a1622
  MSBuild version 17.3.2+561848881 for .NET
    Determining projects to restore...
    Restored /src/HelloWorld.csproj (in 168 ms).
    HelloWorld -> /app/build/HelloWorld.dll

  Build succeeded.
      0 Warning(s)
      0 Error(s)

  Time Elapsed 00:00:06.22
  Removing intermediate container 30866e9a1622
  ---> 38a12007e2a9
  Step 11/16 : FROM build AS publish
  ---> 38a12007e2a9
  Step 12/16 : RUN dotnet publish "HelloWorld.csproj" -c Release -o /app/publish /p:UseAppHost=false
  ---> Running in ce940dbb6ce7
  MSBuild version 17.3.2+561848881 for .NET
    Determining projects to restore...
    All projects are up-to-date for restore.
    HelloWorld -> /src/bin/Release/net6.0/HelloWorld.dll
    HelloWorld -> /app/publish/
  Removing intermediate container ce940dbb6ce7
  ---> 605afdb96a13
  Step 13/16 : FROM base AS final
  ---> abf2dca9ebac
  Step 14/16 : WORKDIR /app
  ---> Running in 900a3c82ac7a
  Removing intermediate container 900a3c82ac7a
  ---> eafc06665d50
  Step 15/16 : COPY --from=publish /app/publish .
  ---> b312648b54f3
  Step 16/16 : ENTRYPOINT ["dotnet", "HelloWorld.dll"]
  ---> Running in 1595a9cd1785
  Removing intermediate container 1595a9cd1785
  ---> 426dc6ef4f07
  Successfully built 426dc6ef4f07
  Successfully tagged helloworld:latest
  SECURITY WARNING: You are building a Docker image from Windows against a non-Windows Docker host. All files and directories added to build context will have '-rwxr-xr-x' permissions. It is recommended to double check and reset permissions for sensitive files and directories.

# Docker Run
  PS C:\K8S-Demo> docker run -p 5000:80  helloworld

  PS C:\K8S-Demo> curl http://localhost:5000/   

  Browse - http://localhost:5000/

# Tag Image
  PS C:\K8S-Demo> docker tag helloworld e880613/helloworld:v1
# Docker Push
  PS C:\K8S-Demo> docker push e880613/helloworld:v1


# Deployment in minikube
  PS C:\K8S-Demo> kubectl apply -f .\Manifest\deployment.yaml         
  deployment.apps/helloworld-deploy created

  PS C:\K8S-Demo> kubectl get deployments
  NAME                READY   UP-TO-DATE   AVAILABLE   AGE
  helloworld-deploy   0/2     2            0           17s 

  PS C:\K8S-Demo> kubectl apply -f .\Manifest\service.yaml 
  service/helloworld-service created

  PS C:\K8S-Demo> minikube service helloworld-service --url

  PS C:\K8S-Demo> minikube service helloworld-service --url
  http://192.168.59.113:30007


# Helm

  helm create helmchart
  helm lint helmchart 
  helm install --dry-run helloworld helmchart/
  helm install  helloworld helmchart/

  helm upgrade  helloworld helmchart/

  PS C:\K8S-Demo> kubectl get all -l "app.kubernetes.io/name=helmchart"
  NAME                                     READY   STATUS    RESTARTS   AGE
  pod/helloworld-helmchart-b48bf9b-2kkp2   1/1     Running   0          14s

  NAME                           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
  service/helloworld-helmchart   NodePort   10.96.48.212   <none>        80:32449/TCP   14s

  NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
  deployment.apps/helloworld-helmchart   1/1     1            1           14s

  NAME                                           DESIRED   CURRENT   READY   AGE
  replicaset.apps/helloworld-helmchart-b48bf9b   1         1         1       14s
  
  PS C:\K8S-Demo> minikube service helloworld-service --url
  http://192.168.59.113:30007