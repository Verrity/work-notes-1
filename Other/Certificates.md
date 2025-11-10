
---
### Теги

### [[ESR|Назад]]
### Далее
####
---

<u>Команды в `WLC-30`</u>:

Генерация сертификатов:
``` unfold bash
update crypto default <cert>
```
Сертификаты ТД лежат в
```unfold 
dir crypto:cert/
```

Удалять сертификаты
```unfold
delete crypto:cert/ECB1E02B3590.pem
```

<u>Вывод сертификатов в разных форматах</u>:
* Для PEM формата (обычно .crt, .pem, .cer)
```unfold
openssl x509 -text -noout -in certificate.crt
```
* Для DER формата (обычно .der, .cer)
```unfold
openssl x509 -inform der -text -noout -in certificate.der
```

* Для PKCS#12 (.p12, .pfx)
```unfold
openssl pkcs12 -in certificate.p12 -nodes | openssl x509 -text -noout
```

* Если в цепочке сертификатов
```unfold
openssl x509 -text -noout -in chain.pem
```