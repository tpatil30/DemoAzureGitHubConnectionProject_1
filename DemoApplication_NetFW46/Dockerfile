#Depending on the operating system of the host machines(s) that will build or run the containers, the image specified in the FROM statement may need to be changed.
#For more information, please see http://aka.ms/containercompat 

FROM microsoft/aspnet:4.7.2
SHELL ["powershell"]

RUN Install-WindowsFeature NET-Framework-45-ASPNET ; \
    Install-WindowsFeature Web-Asp-NET45 ; \
    Add-WindowsFeature Web-Windows-Auth

ARG site_root=.
ADD ${site_root} /inetpub/wwwroot

RUN icacls 'C:\inetpub\wwwroot' /grant 'IIS_IUSRS:(F)' /T

RUN Remove-WebSite -Name 'Default Web Site'; \
    New-WebSite -Name 'DemoApplication_NetFW46' -Port 80 \
    -PhysicalPath 'c:\inetpub\wwwroot' -ApplicationPool 'DemoApplication_NetFW46Pool'; \
    Set-WebConfigurationProperty -filter /system.WebServer/security/authentication/AnonymousAuthentication -name enabled -value false -PSPath IIS:\ ; \
    Set-WebConfigurationProperty -filter /system.webServer/security/authentication/windowsAuthentication -name enabled -value true -PSPath IIS:\ ; \
    Remove-WebConfigurationProperty -filter /system.webServer/defaultDocument -name files.collection -AtElement @{value='iisstart.htm'} ; \
    ('10.64.152.168 SELULASSQLDEV01').trim() |Out-File -Append -FilePath 'C:\Windows\System32\drivers\etc\hosts' -Encoding ascii ;
EXPOSE 80
