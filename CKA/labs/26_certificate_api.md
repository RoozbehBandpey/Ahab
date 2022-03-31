# Certificate API

A new member akshay joined our team. He requires access to our cluster. The Certificate Signing Request is at the /root location.



Inspect it

cat akshay.csr
-----BEGIN CERTIFICATE REQUEST-----
MIICVjCCAT4CAQAwETEPMA0GA1UEAwwGYWtzaGF5MIIBIjANBgkqhkiG9w0BAQEF
AAOCAQ8AMIIBCgKCAQEA5zrngkvLw/H/GVYwPQ0GXXRDOg9Ucw/V3vB3huY2CaiJ
bQbZCs57l2zNKicpf2jA4hxxqSBCV4IsLMWmJYyNHUSFk9tFIDL4cMVbXStPjsJw
I6BUEGSAWqInruaa7QsDMfbExh2yyWsRUj/qUI5HbVRBK8I7AL037AuAPZPHP9Ei
DON8BIvgZl71y0Z97xEEvm+k0RcpHCVUExjYyH37JmA3XnPjInHkbsxnf66X9wqL
tOznskK1HvJqd5918ox8OTrJn54m5ofw2V+yyHWJRCYlLj+FDjLu/C7ZRj68dikC
a79xNLoyVYjQMAh7bGVy0HPmAHImY+GOgd0bFdI23wIDAQABoAAwDQYJKoZIhvcN
AQELBQADggEBAC9NJQmL8Sqyetcys/05rKOVwQ2ienLQEBtfunH8civvJ6dXMZfs
zSGa2/68fkHVVwL5X+5lEkCPQnBQ87iOSMMbOJsH22nY9BYdcqu4CwCVsw1vw0sQ
KTxkmsVaRoB7YpV0cMmlCz0+p8I4eGHOA8XiHKzmf8eXd9DEblg2Up93nNp6k9jg
R2fLzu1r8Fb0guZahCP1/fI0dFuViUekNmrmXPeCtYIr3fwHKxhCCWmfA8ATmUX5
8m5Lgk/F/IfK9mWuIlShCEpN42gx1huCJ0LjiZhik4lFmBmjHCiSm8QL6/HysrRx
R4bElV0rM3Z9KecotxQuUiQXUlXTy0BqEmY=
-----END CERTIFICATE REQUEST-----


Create a CertificateSigningRequest object with the name akshay with the contents of the akshay.csr file



kubectl get csr csr-8zkvm  -o yaml > akshay-csr.yaml
root@controlplane:~# ls
akshay-csr.yaml  akshay.csr  akshay.key

What is the Condition of the newly created Certificate Signing Request object?

Run the command kubectl get csr

Reject that request.

kubectl certificate deny agent-smith

kubectl delete csr agent-smith