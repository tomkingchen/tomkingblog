---
title: 'Setup Splunk Universal Forwarder with TLS'
date: 2021-05-21T18:23:00.005-07:00
draft: false
url: /2021/05/setup-splunk-universal-forwarder-with.html
---

One of the best practice to setup Splunk Universal Forwarder (UF) is to encrypt incoming log traffic with TLS. This is especially important if your intake is from an external source on Internet, e.g from a SaaS solution. In this blog I will demostrate the steps to get this setup. 

  

First, we will create a public A DNS record for the UF. This is because our UF will be receiving logs from Internet. 

  

Next, we need to purchase a new TLS certificate for the A record we just registered. Assume the domain name we set for the certificate is syslog.contoso.com. 

  

On the UF, run the command below to generate a CSR to submit to the public CA (DigiCert, GoDaddy...).

openssl req -new -key syslog.contoso.com.key -out syslog.contoso.com.csr  

  

Copy the private key to Syslog-ng cert.d folder. The private key is automatically generated along with the CSR.

cp syslog.contoso.com.key /usr/local/etc/syslog-ng/cert.d/syslog\_contoso\_com\_key.pem  

  

Submit the CSR file to CA and wait for the certificate to be issued. Once you receive the certificate, upload it to the UF server and move them to Syslog-ng cert.d folder. 

mv syslog\_contoso\_com.crt /usr/local/etc/syslog-ng/cert.d/syslog\_contoso\_com.pem

  

Once you have the certificate and its private key in place, add following code to Syslog-ng's conf file. You can normally find the config file at /etc/syslog-ng/conf.d/syslog-ng.conf.

source contoso\_saasapp\_syslog {

network(

ip(0.0.0.0)

port(1999)

transport("tls")

tls(

key-file("/usr/local/etc/syslog-ng/cert.d/syslog\_contoso\_com\_key.pem")

cert-file("/usr/local/etc/syslog-ng/cert.d/syslog\_contoso\_com.pem")

peer-verify(optional-untrusted)

)

);

};

  

Restart Splunk service and the UF should now accept logs with TLS.