NAMESPACE := graylog
ELASTICSEARCH_IMAGE := docker.elastic.co/elasticsearch/elasticsearch:8.5.1

TIMEOUT := 1200s

install:
	helm upgrade --wait --timeout=$(TIMEOUT) --install --values values.yaml

purge:
	kubectl -n $(NAMESPACE) delete secrets elastic-certificates elastic-certificate-pem elastic-certificate-crt|| true
	helm del $(RELEASE)

secrets:
	docker rm -f elastic-helm-charts-certs || true
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-certificate.crt elastic-stack-ca.p12 || true
	docker run --name elastic-helm-charts-certs -i -w /tmp \
		$(ELASTICSEARCH_IMAGE) \
		/bin/sh -c " \
			elasticsearch-certutil ca --out /tmp/elastic-stack-ca.p12 --pass '' && \
			elasticsearch-certutil cert --name security-master --dns security-master --ca /tmp/elastic-stack-ca.p12 --pass '' --ca-pass '' --out /tmp/elastic-certificates.p12" && \
	docker cp elastic-helm-charts-certs:/tmp/elastic-certificates.p12 ./ && \
	docker rm -f elastic-helm-charts-certs && \
	openssl pkcs12 -nodes -passin pass:'' -in elastic-certificates.p12 -out elastic-certificate.pem && \
	openssl x509 -outform der -in elastic-certificate.pem -out elastic-certificate.crt && \
	kubectl -n $(NAMESPACE) create secret generic elastic-certificates --from-file=elastic-certificates.p12 && \
	kubectl -n $(NAMESPACE) create secret generic elastic-certificate-pem --from-file=elastic-certificate.pem && \
	kubectl -n $(NAMESPACE) create secret generic elastic-certificate-crt --from-file=elastic-certificate.crt && \
	rm -f elastic-certificates.p12 elastic-certificate.pem elastic-certificate.crt elastic-stack-ca.p12