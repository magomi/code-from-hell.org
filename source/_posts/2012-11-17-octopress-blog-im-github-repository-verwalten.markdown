---
layout: post
title: "Octopress-Blog im Github-Repository verwalten"
date: 2012-11-17 14:07
comments: true
author: Marco
categories: ["octopress", "github"]
---
Das gesamte Blog kann an sich ohne weiteres in einem Github-Repository verwaltet werden:

  *   neues Repository auf GitHub einrichten,
  *   im lokalen Octopress-Verzeichnis die URL des Repositories setzten ("git init && git config remote.origin.url <url.des.github.repo>"),
  *   den initialen Stand commiten ("git add . && git commit -a -m '<commit.message>'"),
  *   den aktuellen Stand auf den Server pushen ("git push origin master:master").

In den Admin-Einstellungen des Repositories auf GitHub kann man andere Nutzer mit Schreibberechtigung hinzufügen. Fertig ist das Mehrbenutzer-Blog.

Das Deployment auf dem Zielserver kann auch direkt aus dem github-Repository erfolgen. Leider geht das nicht ganz so komfortabel, wie wenn man das Blog direkt via <a href="http://octopress.org/docs/deploying/github/">github-Pages betreibt</a>. Der eigentliche Inhalt des Blogs liegt im Unterverzeichnis "public". Grundsätzlich würde es also ausreichen, dieses Unterverzeichnis auszuchecken. Derartige sparse clones werden von git nicht unterstützt. Bleibt also nur, das gesamte Repository auf dem Zielserver auszuchecken und den Webserver nur auf das public Verzeichnis verweisen zu lassen. Einer der großen Vorteile, den Blog auf github-Pages zu betreiben liegt darin, dass bei einem push ins Repository auch gleich der Blogcontent aktualisiert wird. Mit einem kleinen Skript und einem Github-Hook lässt sich das aber recht einfach auf dem eigenen Server nachbauen. Man nehme:

  *   ein Skript, dass ins lokale Repository auf dem Webserver wechselt und via "git pull..." den aktuellen Stand auscheckt

     {% codeblock update_code-from-hell.org.sh lang:bash %}
#!/bin/sh
cd /home/serveradmin/website/codefromhell/
git pull --rebase
cd -
     {% endcodeblock %}

  *   einen Mini-Webserver, der von einem Github-Hook angesprochen werden kann und das update-Skript aufruft

     {% codeblock httpsrv.rb lang:ruby %}
require 'socket'

server = TCPServer.new(2000)

puts "#{Time.now} - listener started"

while (session = server.accept)
   session.print "HTTP/1.1 200 OK\r\n#{Time.now}\r\nServer: UpdateNotificationListener\r\nContent-Type: text/html\r\n\r\n"
   requestPars = session.gets.chomp.split
   requestMethod = requestPars[0]
   requestUrl = requestPars[1]
   requestProt = requestPars[2]
   puts "#{Time.now} - request recieved: #{requestMethod} #{requestUrl} #{requestProt}"
   if requestMethod == "POST" && requestUrl == "/checkForNewVersion"
       puts "#{Time.now} - start update of the repository..."
       system('/home/serveradmin/website/updater/update_code-from-hell.org.sh')
       puts "#{Time.now} - ...update done"
   end 
   session.print "<html>thanks, bye</html>"
   session.close
end
     {% endcodeblock %}

  *   einen github-WebHook, der auf die passende URL zeigt:

      {% img /images/code-from-hell/webhook.png 'github WebHook' %}

