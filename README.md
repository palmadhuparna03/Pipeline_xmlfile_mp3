# Pipeline_xmlfile_mp3
<?xml version="1.0" encoding="UTF-8"?>
<pipeline>
  <!-- Pipeline Configuration -->
  <metadata>
    <name>Build Pipeline</name>
    <version>1.0</version>
    <description>Automated build and deployment pipeline</description>
    <createdDate>2026-03-10</createdDate>
  </metadata>

  <!-- Environment Configuration -->
  <environments>
    <environment name="development">
      <branch>develop</branch>
      <autoTrigger>true</autoTrigger>
    </environment>
    <environment name="staging">
      <branch>staging</branch>
      <autoTrigger>true</autoTrigger>
      <requiresApproval>false</requiresApproval>
    </environment>
    <environment name="production">
      <branch>main</branch>
      <autoTrigger>false</autoTrigger>
      <requiresApproval>true</requiresApproval>
      <approvers>
        <approver>tech-lead</approver>
        <approver>devops-team</approver>
      </approvers>
    </environment>
  </environments>

  <!-- Pipeline Stages -->
  <stages>
    <!-- Stage 1: Checkout -->
    <stage name="Checkout" order="1">
      <description>Clone repository and prepare workspace</description>
      <tasks>
        <task name="Clone Repository">
          <action type="git-clone">
            <repository>https://github.com/organization/repository.git</repository>
            <branch>${GIT_BRANCH}</branch>
          </action>
        </task>
      </tasks>
      <timeout>300</timeout>
    </stage>

    <!-- Stage 2: Build -->
    <stage name="Build" order="2">
      <description>Compile and build application</description>
      <tasks>
        <task name="Install Dependencies">
          <action type="shell">
            <command>npm install</command>
          </action>
        </task>
        <task name="Build Application">
          <action type="shell">
            <command>npm run build</command>
          </action>
        </task>
      </tasks>
      <timeout>600</timeout>
      <onFailure>ABORT</onFailure>
    </stage>

    <!-- Stage 3: Test -->
    <stage name="Test" order="3">
      <description>Run unit and integration tests</description>
      <tasks>
        <task name="Unit Tests">
          <action type="shell">
            <command>npm test -- --coverage</command>
          </action>
        </task>
        <task name="Integration Tests">
          <action type="shell">
            <command>npm run test:integration</command>
          </action>
        </task>
      </tasks>
      <timeout>900</timeout>
      <onFailure>ABORT</onFailure>
      <reports>
        <report type="junit" path="test-results/**/*.xml</report>
        <report type="coverage" path="coverage/coverage-final.json</report>
      </reports>
    </stage>

    <!-- Stage 4: Code Quality -->
    <stage name="Code Quality" order="4">
      <description>Analyze code quality and security</description>
      <tasks>
        <task name="SonarQube Analysis">
          <action type="sonar">
            <projectKey>my-project</projectKey>
            <sources>src</sources>
            <qualityGate>true</qualityGate>
          </action>
        </task>
        <task name="SAST Security Scan">
          <action type="security-scan">
            <scanType>sast</scanType>
            <failOnCritical>true</failOnCritical>
          </action>
        </task>
      </tasks>
      <timeout>600</timeout>
      <onFailure>WARN</onFailure>
    </stage>

    <!-- Stage 5: Build Artifact -->
    <stage name="Build Artifact" order="5">
      <description>Create and push Docker image</description>
      <tasks>
        <task name="Build Docker Image">
          <action type="docker-build">
            <dockerfile>Dockerfile</dockerfile>
            <imageName>myapp</imageName>
            <imageTag>${BUILD_NUMBER}</imageTag>
            <registry>docker.io/organization</registry>
          </action>
        </task>
        <task name="Push to Registry">
          <action type="docker-push">
            <image>docker.io/organization/myapp:${BUILD_NUMBER}</image>
          </action>
        </task>
      </tasks>
      <timeout>1200</timeout>
      <onFailure>ABORT</onFailure>
    </stage>

    <!-- Stage 6: Deploy to Dev -->
    <stage name="Deploy Dev" order="6">
      <description>Deploy to development environment</description>
      <tasks>
        <task name="Deploy Application">
          <action type="kubernetes-deploy">
            <namespace>development</namespace>
            <deployment>myapp-deployment</deployment>
            <image>docker.io/organization/myapp:${BUILD_NUMBER}</image>
          </action>
        </task>
      </tasks>
      <timeout>600</timeout>
      <onFailure>ABORT</onFailure>
      <environment>development</environment>
    </stage>

    <!-- Stage 7: Deploy to Staging -->
    <stage name="Deploy Staging" order="7">
      <description>Deploy to staging environment</description>
      <tasks>
        <task name="Deploy Application">
          <action type="kubernetes-deploy">
            <namespace>staging</namespace>
            <deployment>myapp-deployment</deployment>
            <image>docker.io/organization/myapp:${BUILD_NUMBER}</image>
          </action>
        </task>
        <task name="Run Smoke Tests">
          <action type="shell">
            <command>npm run test:smoke -- --environment=staging</command>
          </action>
        </task>
      </tasks>
      <timeout>900</timeout>
      <onFailure>ABORT</onFailure>
      <environment>staging</environment>
    </stage>

    <!-- Stage 8: Deploy to Production -->
    <stage name="Deploy Production" order="8">
      <description>Deploy to production environment</description>
      <tasks>
        <task name="Blue-Green Deployment">
          <action type="kubernetes-deploy">
            <namespace>production</namespace>
            <deployment>myapp-deployment</deployment>
            <image>docker.io/organization/myapp:${BUILD_NUMBER}</image>
            <strategy>blue-green</strategy>
            <healthCheck>
              <endpoint>https://myapp.com/health</endpoint>
              <timeout>300</timeout>
            </healthCheck>
          </action>
        </task>
      </tasks>
      <timeout>1200</timeout>
      <onFailure>ROLLBACK</onFailure>
      <environment>production</environment>
      <approval>true</approval>
    </stage>

    <!-- Stage 9: Notification -->
    <stage name="Notification" order="9">
      <description>Send pipeline completion notifications</description>
      <tasks>
        <task name="Notify Team">
          <action type="notification">
            <channels>
              <channel type="slack">
                <webhook>https://hooks.slack.com/services/YOUR/WEBHOOK/URL</webhook>
                <message>Pipeline ${PIPELINE_NAME} completed with status: ${PIPELINE_STATUS}</message>
              </channel>
              <channel type="email">
                <recipients>
                  <recipient>team@organization.com</recipient>
                </recipients>
              </channel>
            </channels>
          </action>
        </task>
      </tasks>
      <timeout>60</timeout>
      <runAlways>true</runAlways>
    </stage>
  </stages>

  <!-- Variables and Credentials -->
  <variables>
    <variable name="BUILD_NUMBER" type="system" />
    <variable name="GIT_BRANCH" type="system" default="main" />
    <variable name="GIT_COMMIT" type="system" />
    <variable name="REGISTRY_URL" type="environment" default="docker.io" />
  </variables>

  <credentials>
    <credential name="git-credentials" type="ssh-key">
      <username>git</username>
      <privateKeyPath>${CREDENTIALS_PATH}/git-key</privateKeyPath>
    </credential>
    <credential name="docker-registry" type="registry">
      <username>${DOCKER_USERNAME}</username>
      <password>${DOCKER_PASSWORD}</password>
    </credential>
    <credential name="kubernetes" type="kubeconfig">
      <configPath>${KUBECONFIG_PATH}</configPath>
    </credential>
  </credentials>

  <!-- Triggers -->
  <triggers>
    <trigger name="SCM Webhook">
      <type>webhook</type>
      <event>push</event>
      <branches>
        <branch>main</branch>
        <branch>develop</branch>
        <branch>staging</branch>
      </branches>
    </trigger>
    <trigger name="Scheduled Build">
      <type>cron</type>
      <schedule>0 2 * * *</schedule>
      <description>Daily build at 2 AM</description>
    </trigger>
  </triggers>

  <!-- Notifications -->
  <notifications>
    <notification event="PIPELINE_SUCCESS">
      <channels>
        <slack>Build succeeded!</slack>
      </channels>
    </notification>
    <notification event="PIPELINE_FAILURE">
      <channels>
        <slack>Build failed! Check logs immediately.</slack>
        <email>devops@organization.com</email>
      </channels>
    </notification>
  </notifications>

</pipeline>
