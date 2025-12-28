pipeline{
 agent{
  node{
  label "agent"
  }
 }
 parameters {
        string(name: 'PERSON', defaultValue: 'Mr Chakradhar', description: 'Who should I say hello to?')
        booleanParam(name: 'Deploy', defaultValue: false, description: 'Toggle this value')
 }
 environment{
 course = "jenkins"
 apiVersion = ""
 acc_id = "406682759639"
 project = "chakra"
 component = "catalogue"
 }
 options{
  disableConcurrentBuilds()
 }
 stages{
    stage('Read json file'){
    steps{
     script{
     def packagejson = readJSON file: "package.json"
     apiVersion = packagejson.version
     echo "apiversion: ${apiVersion}"
     }
    }
  }
/*
  stage('Sonar Code Analysis') {
   environment{
   def scannerHome = tool 'sonar-8.0'
   }
   steps {
                script {
                    withSonarQubeEnv('sonar-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
   }
  }
  stage('Quality Gate') {
   steps {
      // Wait for Quality Gate, fail build if needed
      timeout(time: 1, unit: 'HOURS') {
        waitForQualityGate abortPipeline: true
      }
   }
  } */
  stage('Dependabot Security Gate') {
            environment {
                GITHUB_OWNER = 'Chakradhar-Jeereddy'
                GITHUB_REPO  = 'practise'
                GITHUB_API   = 'https://api.github.com'
                GITHUB_TOKEN = credentials('GITHUB_TOKEN')
            }

            steps {
                script{
                    /* Use sh """ when you want to use Groovy variables inside the shell.
                    Use sh ''' when you want the script to be treated as pure shell. */
                    sh '''
                    echo "Fetching Dependabot alerts..."

                    response=$(curl -s \
                        -H "Authorization: token ${GITHUB_TOKEN}" \
                        -H "Accept: application/vnd.github+json" \
                        "${GITHUB_API}/repos/${GITHUB_OWNER}/${GITHUB_REPO}/dependabot/alerts?per_page=100")

                    echo "${response}" > dependabot_alerts.json

                    high_critical_open_count=$(echo "${response}" | jq '[.[] 
                        | select(
                            .state == "open"
                            and (.security_advisory.severity == "high"
                                or .security_advisory.severity == "critical")
                        )
                    ] | length')

                    echo "Open HIGH/CRITICAL Dependabot alerts: ${high_critical_open_count}"

                    if [ "${high_critical_open_count}" -gt 0 ]; then
                        echo "❌ Blocking pipeline due to OPEN HIGH/CRITICAL Dependabot alerts"
                        echo "Affected dependencies:"
                        echo "$response" | jq '.[] 
                        | select(.state=="open" 
                        and (.security_advisory.severity=="high" 
                        or .security_advisory.severity=="critical"))
                        | {dependency: .dependency.package.name, severity: .security_advisory.severity, advisory: .security_advisory.summary}'
                        exit 1
                    else
                        echo "✅ No OPEN HIGH/CRITICAL Dependabot alerts found"
                    fi
                    '''
                    
                }
            }
  }
  stage('Build catalogue image'){
    steps{
      withAWS(region:'us-east-1',credentials:'aws-ecr') {
      sh"""
      aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${acc_id}.dkr.ecr.us-east-1.amazonaws.com
      docker build -t ${acc_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${apiVersion} .
      docker push ${acc_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${apiVersion}
      """
      }
    }
  }
  stage('Test'){
    environment{
    name = "chakras testing"
    }
    steps{
    echo "Testing"
    echo "${name}"
    }
  }
  stage('Deploy'){
    when{
    expression { params.Deploy == "true" }
    }
    steps{
    echo "Deploying"
    }
  }
 }
 post{
  always{
   cleanWs()
   echo "Always say hi"
  }
  success{
  echo "Passed the deployment"
  }
  failure{
  echo "Deployment failed"
  }
  aborted{
  echo "deployment aborted"
  }
 }
}
