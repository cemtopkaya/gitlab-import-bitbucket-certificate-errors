# GitLab'ın Kurulumu

Ayaklandırıp root kullanıcısının şifresini alalım:

```shell
docker-compose up -d
docker exec -it git grep 'Password:' /etc/gitlab/initial_root_password
```

![](.vscode/readme-images/2022-08-21-16-32-31.png)

![](.vscode/readme-images/2022-08-21-18-45-35.png)

Harici kaynaklardan kod havuzlarını çekebilmek için aşağıdaki ayarı (Admin Area > Network > Outbound requests > "Allow requests to the local network from web hooks and services") yapıyoruz:

![](.vscode/readme-images/2022-08-21-16-35-24.png)

![](.vscode/readme-images/2022-08-21-18-46-31.png)

# Yeni Proje (Bitbucket'tan İçeri Alacağız)

![](.vscode/readme-images/2022-08-21-16-37-36.png)

![](.vscode/readme-images/2022-08-21-16-38-34.png)

![](.vscode/readme-images/2022-08-21-16-39-10.png)

# Bitbucket Kod Havuzları

https://bitbucket.ulakhaberlesme.com.tr:8443/rest/api/1.0/repos?start=0&limit=25 Adresindeki listeye erişmek birinci gaye.
Bu sayfadan alınacak listeden seçtiğimiz kod havuzlarını Gitlab içine alacağız.

![](.vscode/readme-images/2022-08-21-16-43-30.png)

![](.vscode/readme-images/2022-08-21-16-45-40.png)

# Bitbucket Üstünde Kullanıcı Token'ı Oluşturmak

https://bitbucket.ulakhaberlesme.com.tr:8443/plugins/servlet/access-tokens/manage Adresinden TOKEN oluşturacağız.

![](.vscode/readme-images/2022-08-21-16-41-54.png)

![](.vscode/readme-images/2022-08-21-16-40-22.png)

![](.vscode/readme-images/2022-08-21-16-46-17.png)

![](.vscode/readme-images/2022-08-21-17-28-08.png)


# Hatanın Tespiti

Unable to connect to server: SSL_connect returned=1 errno=0 state=error: certificate verify failed (unable to get local issuer certificate)

![](.vscode/readme-images/2022-08-21-19-23-05.png)

```shell
curl https://bitbucket.ulakhaberlesme.com.tr:8443/repos?visibility=public
```

![](.vscode/readme-images/2022-08-21-18-51-32.png)


```shell
echo | /opt/gitlab/embedded/bin/openssl s_client -connect bitbucket.ulakhaberlesme.com.tr:8443
```

![](.vscode/readme-images/2022-08-21-18-52-41.png)

Çözümü:

Sertifikaları aşağıda anlatıldığı gibi 


![](.vscode/readme-images/2022-08-21-16-21-56.png)

Sorun giderildikten sonra:

![](.vscode/readme-images/2022-08-21-16-20-46.png)

## Firefox'tan Sertifikaların İndirilmesi

### Tüm Sertifikaların Zincir Olarak İndirilmesi ve curl İle Denenmesi

Önce sertifika zincirini tek dosya olarak indirip `/tmp/ulakhaberlesme-com-tr-chain.pem` olarak kaydedelim ve curl ile bir istek yapalım.

![](.vscode/readme-images/2022-08-21-19-06-20.png)

```shell
curl --cacert /tmp/ulakhaberlesme-com-tr-chain.pem https://bitbucket.ulakhaberlesme.com.tr:8443/rest/api/1.0/repos?start=0&limit=25
```

![](.vscode/readme-images/2022-08-21-19-07-18.png)

### Tüm Sertifikaları tek tek İndirilip ve openssl İle Denenmesi

Sertifikaların PEM hallerini indirip `/etc/gitlab/trusted-certs` dizinine yapıştıracağız.

![](.vscode/readme-images/2022-08-21-15-46-21.png)

![](.vscode/readme-images/2022-08-21-16-10-55.png)

![](.vscode/readme-images/2022-08-21-16-11-56.png)

![](.vscode/readme-images/2022-08-21-16-13-43.png)

![](.vscode/readme-images/2022-08-21-16-14-35.png)

![](.vscode/readme-images/2022-08-21-16-15-15.png)

![](.vscode/readme-images/2022-08-21-16-15-49.png)

Dosyaları `/etc/gitlab/trusted-certs` dizinine kopyalayım:

![](.vscode/readme-images/2022-08-21-19-14-29.png)

Dosyalar yapıştırıldı ama gitlab bu dosyaları taramadığı için henüz çalışmıyor:

```shell
echo | /opt/gitlab/embedded/bin/openssl s_client -connect bitbucket.ulakhaberlesme.com.tr:8443
```

![](.vscode/readme-images/2022-08-21-19-16-27.png)

Gitlab'ı tekrar ayarlarla başlatıp tekrar deneyeceğiz:

```shell
gitlab-ctl reconfigure
```

Çıktılarına baktığımızda sertifikaların gitlab tarafından işlendiğini görebiliriz:

![](.vscode/readme-images/2022-08-21-19-18-31.png)

Gitlab tekrar ayarlandığına göre openssl ile tekrar deneyebiliriz:

![](.vscode/readme-images/2022-08-21-19-19-06.png)

```shell
echo | /opt/gitlab/embedded/bin/openssl s_client -connect bitbucket.ulakhaberlesme.com.tr:8443
```

![](.vscode/readme-images/2022-08-21-19-20-44.png)



# Bitbucket'tan Repo'ların Çekilmesi

![](.vscode/readme-images/2022-08-21-19-22-19.png)

![](.vscode/readme-images/2022-08-21-16-19-22.png)
