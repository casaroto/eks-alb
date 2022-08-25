

#Crie o cluster EKS seguindo o template: Deploy Amazon EKS into a new VPC
#arquivo amazon-eks-entrypoint-new-vpc.template.yaml
#doc: https://aws-quickstart.github.io/quickstart-amazon-eks/#_launch_the_quick_start

#utilize este exemplo para subir um service com as anootations exemplo
#utilize alb.ingress.kubernetes.io/scheme: internet-facing 
#para primeiro testar direto sem api Gateway
#depois utilize alb.ingress.kubernetes.io/scheme: internal
#assim deixará só disponível depois via api Gateway
kubectl apply -f nginx-deploy.yaml

#esta doc explica o que deve ter no exemplo de app com gateway
#https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html


#passos simplificados para criação do gateway
aws apigatewayv2 create-api \
    --name my-http-api \
    --protocol-type HTTP

#Pegue no cluster eks as subnets utilzadas e os security groups
aws apigatewayv2 create-vpc-link --name MyVpcLink \
    --subnet-ids subnet-066256ae0ff57e3b7 subnet-04514d7db761fcfcc \
    --security-group-ids sg-07282bfd4eea405d6 sg-0adc0c3e2baf61704

#pegue o id da http api criada e coloque em --api-id 
#pegue o id vpc link criada e coloque em --connection-id 
aws apigatewayv2 create-integration --api-id glfiulmpwd --integration-type HTTP_PROXY \
    --integration-method GET --connection-type VPC_LINK \
    --connection-id gs9rry \
    --integration-uri arn:aws:elasticloadbalancing:us-east-2:361353278934:listener/app/k8s-game2048-ingress2-244f561ef3/200b2f522abbb8f4/b84313b4266653d0 \
    --payload-format-version 1.0

#documentação explicativa sobre criação dos objetos do gateway
#https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-vpc-links.html
#https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-develop-integrations-private.html

#Em Api Gateway abra a http api criada 
#crie a rota que quer mapear e ligue a integration criada
#edite a integration, crie um Parameter mapping nela com esta configuração:

\#  Parameter to modify  Modification type   Value

\#    path       overwrite      $request.path

#crie um stage com o nome $default para fazer o deploy
#teste a url gerada no stage, exemplo: Invoke URL: https://glfiulmpwd.execute-api.us-east-2.amazonaws.com