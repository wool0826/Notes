이것저것 찾아보다가 [2023년 웹 개발 트렌드](https://www.youtube.com/watch?v=clbI-NL4s2k) 라는 영상을 보고 재밌는 내용인 것 같아서 정리를 해보았습니다.

# Terraform

IaC(Infrastructure as Code) Tool 중 하나입니다.
다양한 Provider(AWS, Azure, GCP, NCP, ...)의 리소스를 파일로 관리할 수 있게 해줍니다.

기술이 나온지는 꽤 되었는데요. 2014년에 나왔는데, 최근에 사용량이 많이 늘어나고 있는 것으로 보이네요!

## 주요 개념

HCL(HashiCorp Configuration Language) 이나 JSON 을 사용해서 Infrastructure에서 사용할 리소스를 정의하고 배포/수정/삭제를 할 수 있습니다.
보다보니 뭔가 Helm과 되게 유사해보인다는 생각이 들었는데요. 기존에 알고 있는 helm과 비교하면 이해가 쉬울 것 같아서, 간단히 비교하면서 확인해보도록 하겠습니다.

### Helm 과의 비교
> https://phoenixnap.com/blog/helm-vs-terraform

#### 공통점

- Kubernetes 리소스를 코드로 정의하고 관리할 수 있게 해줍니다.
- dry-run 모드가 존재하여, 변경사항이 반영되었을 때의 결과를 확인할 수 있도록 도와줍니다.
- 환경변수를 사용할 수 있습니다.
- 다양한 provider가 존재하여, 여러 곳에서 써먹을 수 있습니다.

#### 차이점

- Terraform은 Kubernetes 만 관리하는 것이 아니라, 다양한 Provider의 리소스를 동시에 관리가 가능합니다.
- Terraform은 Kubernetes 리소스뿐만 아니라 클러스터에 대한 관리도 가능합니다. (helm은 리소스만 관리가능)
    - 즉, 하나의 tool 로 인프라/클러스터 매니징을 동시에 할 수 있다는 것을 의미합니다.
    - (Kubernetes 는 Terraform provider 중 하나이므로 어찌보면 당연한 것으로 볼 수도 있겠네요.)
- Terraform은 Kubernetes 클러스터에 아무것도 설치하지 않아도 됩니다. (helm 은 Tiller Server의 설치가 필요)
    - 어떻게 보면 당연한 것 같은게, K8s 클러스터를 관리해야하는데, 클러스터에 Terraform 관련 서비스를 설치해둬야된다가 앞뒤가 맞지 않게 되는 것 같습니다.
    - 이건 단점이기도 한데, Terraform은 실행되고 있는 팟에 대한 관리는 할 수 없다고 하네요. Helm 은 Tiller Server K8s API를 호출할 수 있다고 합니다!
- Terraform은 환경변수를 사용할 수 있습니다. (helm 은 제한적이라고 하네요)
- Terraform은 아직 신생(...?) 기능이라서 안정적인 helm 에 비해 지원되지 않는 기능이 많습니다.
- Terraform은 Rollback 이 안 된다고 합니다. 즉, 롤백이 필요하면 기존 리소스를 제거하고 다시 만드는 식으로 진행해야한다고 합니다.
    - 요건 꽤 큰 단점같은데, 뭔가 개선방법이 없을지 아니면 제가 잘못 찾은 건지 좀 궁금하긴 하네요.

위 URL 에서는 Terraform 으로 Helm 을 관리하고, Helm 으로 Kubernetes resource 를 관리하라는 식의 이야기를 하는 것을 보면, 어느정도 답은 정해져있던 것 같습니다.
Terraform 이 뭔가 한 단계 더 큰 개념인 것으로 보이네요. Kubernetes 레벨에 더 가까운 것 같습니다.

## 간단 사용법

Kubernetes 도 비슷한 것 같은데요. "최상의 상태" 를 선언해두면, terraform 이 알아서 해당 상태를 유지하려는 방향으로 동작하게 됩니다.

- terraform init
    - 설정 디렉토리를 생성
- terraform plan
    - 선언한 리소스를 반영했을 때, 어떻게 변경이 일어나는지 확인
- terraform apply
    - "최상의 상태"에 대한 정보를 전달함.
- terraform destroy
    - 리소스 제거

이 모든 작업이 .tf 파일을 통해서 관리되게 되는데요. 이런 걸 보면, 이전에 만기님이 정리해주신 [gitops](https://yobi.navercorp.com/TechTrending_Meetup/posts/49?referrerId=-693805221) 와 유사하게 해당 변경사항이 머지되면 자동으로 배포되게 하는 식으로 구성할 수도 있겠다는 생각이 들었습니다.

코드는 간단히 훑어보고 넘어가면 될 것 같습니다!
resource의 type은 각각의 provider 에서 제공해주고 있으니, 필요한 경우 확인해서 작업을 진행하면 될 것 같습니다.

### AWS

지금은 편의를 위해 하나의 파일로 보이는 것처럼 작성이 되어있는데요.
Terraform 에서는 .tf 파일을 모두 조회한 다음 작업을 진행하기 때문에, 개발자의 필요에 따라 파일로 구분이 가능합니다.

~~~
// 테라폼 메인 설정
terraform {
  required_version = "~>0.12"
  backend "remote" {
    organization = "datapunks"
    workspaces {
      name = "iac_aws_cluster"
    }
  }
}

// 어떤 provider 를 이용할지 선언
provider "aws" {
  region  = "ap-northeast-2"
}

// resource 를 만들기 위한 data 를 선언
data "aws_ami" "coreos" {
  most_recent = true

  filter {
    name   = "name"
    values = ["*CoreOS*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

// resource 생성. 여기서는 aws EC2 instance를 생성
// "aws_instance" 라는 리소스 타입은 AWS 에서 정의한 타입. Terraform 은 선언해놓은대로 provider 에 API 호출을 해서 리소스를 관리한다.
resource "aws_instance" "coreos" {
    count = 2
    
    // 위에서 선언한 data를 사용. 선언의 순서는 관계없이, Terraform 이 알아서 의존관계를 분석해서 리소스를 순차적으로 배포하거나 한다.
    ami = "${data.aws_ami.coreos.id}" 
    instance_type = "t2.micro"
    vpc_security_group_ids = ["${aws_security_group.ec2_front_end.id}"]
  
    tags { 
  	  Name = "example" 
    }
}
~~~

### K8s without Helm
> https://cwal.tistory.com/89

커스텀리소스는 바로 사용할 수는 없고 우회를 통해 관리할 수 있다고 하는데, 조금 불편할 수 있을 것 같긴 하네요.

~~~
provider "kubernetes" {
  host = "https://cluster_endpoint:port"

  client_certificate     = file("~/.kube/client-cert.pem")
  client_key             = file("~/.kube/client-key.pem")
  cluster_ca_certificate = file("~/.kube/cluster-ca-cert.pem")
}

// namespace 에 대한 resource 선언
resource "kubernetes_namespace" "example" {
  metadata {
    name = "my-first-namespace"
  }
}

// deployment 에 대한 resource 선언
resource "kubernetes_deployment" "test" {
  metadata {
    name = "test"
    namespace= kubernetes_namespace.example.metadata.0.name
  }
  // deployment.yaml 에서 표현되는 스펙들을 유사하게 적어주면 되는 것으로 보입니다.
  spec {
    replicas = 2
    selector {
      match_labels = {
        app = "test"
      }
    }
    template {
      metadata {
        labels = {
          app  = "test"
        }
      }
      spec {
        container {
          image = "nginx:1.19.4"
          name  = "nginx"

          resources {
            limits = {
              memory = "512M"
              cpu = "1"
            }
            requests = {
              memory = "256M"
              cpu = "50m"
            }
          }
        }
      }
    }
  }
}
~~~

### K8s with Helm

~~~
provider "helm" {
  // provider "kubernetes" 를 감싸고 있는 방식
  kubernetes {
    host     = "https://cluster_endpoint:port"

    client_certificate     = file("~/.kube/client-cert.pem")
    client_key             = file("~/.kube/client-key.pem")
    cluster_ca_certificate = file("~/.kube/cluster-ca-cert.pem")
  }
}

resource "helm_release" "example" {
  name       = "my-redis-release"
  repository = "https://charts.bitnami.com/bitnami"
  chart      = "redis"
  version    = "6.0.1"

  values = [
    "${file("values.yaml")}"
  ]

  set {
    name  = "cluster.enabled"
    value = "true"
  }

  set {
    name  = "metrics.enabled"
    value = "true"
  }

  set {
    name  = "service.annotations.prometheus\\.io/port"
    value = "9127"
    type  = "string"
  }
}
~~~