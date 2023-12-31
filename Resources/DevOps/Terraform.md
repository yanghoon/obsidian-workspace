#terraform #localstack
**Docker Compose**

```yaml
version: '3.8'

# https://hub.docker.com/r/hashicorp/terraform/tags?page=1&name=1.6
services:
  terraform:
    image: hashicorp/terraform:1.6.3
    entrypoint: tail -f /dev/null
    volumes:
      - ./terraform-infra:/infra
    working_dir: /infra
    environment:
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
      - AWS_SESSION_TOKEN=${AWS_SESSION_TOKEN}
```

## Basics

### Variables

### Module - AWS Provider

### Tools

#### [Pluralith](https://docs.pluralith.com/)

#### LocalStack
* [GitHub](https://github.com/localstack/localstack/tree/master)
* [docker-compose.yaml](https://github.com/localstack/localstack/blob/master/docker-compose.yml)
* links
	* [LocalStack 을 활용한 AWS 환경 구축](https://velog.io/@csh0034/LocalStack-%EC%9D%84-%ED%99%9C%EC%9A%A9%ED%95%9C-AWS-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95)
## References


* Docs
	* [Install Terraform](https://developer.hashicorp.com/terraform/downloads)
* Best Practices
	* [Terraform + Ansible 로 한번에 배포하기](https://kevin-park.medium.com/terraform-ansible-%EB%A1%9C-%ED%95%9C%EB%B2%88%EC%97%90-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-713f719a2433)
	* [Terraform을 사용하여 AWS에서 IAM 사용자 생성](https://ko.linux-console.net/?p=3405)
* Docker Compose
	* [HOW TO USE TERRAFORM VIA DOCKER COMPOSE FOR PROS](https://londonappdeveloper.com/how-to-use-terraform-via-docker-compose-for-professional-developers/ "Permanent Link: How to use Terraform via Docker Compose for Pros")