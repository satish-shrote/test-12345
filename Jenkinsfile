pipeline {
  agent any

  environment {
    ACR_NAME = 'devopsacrrg'  // your ACR name (no .azurecr.io)
    IMAGE_NAME = 'hello-app'
    TAG = 'v1'
  }

  stages {
    stage('Login to Azure') {
      steps {
        withCredentials([string(credentialsId: 'azure-sp', variable: 'AZURE_CREDENTIALS')]) {
          writeFile file: 'azureAuth.json', text: "${AZURE_CREDENTIALS}"
          sh '''
            az login --service-principal \
              --username $(jq -r .clientId azureAuth.json) \
              --password $(jq -r .clientSecret azureAuth.json) \
              --tenant $(jq -r .tenantId azureAuth.json)

            az account set --subscription $(jq -r .subscriptionId azureAuth.json)
          '''
        }
      }
    }

    stage('Build & Push Image') {
      steps {
        sh '''
          az acr login --name $ACR_NAME

          docker build -t $ACR_NAME.azurecr.io/$IMAGE_NAME:$TAG .
          docker push $ACR_NAME.azurecr.io/$IMAGE_NAME:$TAG
        '''
      }
    }

    stage('Deploy to AKS') {
      steps {
        sh '''
          az aks get-credentials --resource-group <your-resource-group> --name <your-aks-name>

          kubectl set image deployment/hello-deployment hello-container=$ACR_NAME.azurecr.io/$IMAGE_NAME:$TAG
        '''
      }
    }
  }
}

