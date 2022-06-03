@NonCPS
def getBBsToDeploy (projectName) {
  if (projectName == "OGW")
    return "o2auws, ogws, ogwf, ogeg, o2abin, ogdrools, ogsearch, o2adb, ogopui, o2acms"
  else if (projectName == "O2A")
    return "o2abin, o2adb, o2aportal, o2awf, o2ataskui, o2aws"
  else if (projectName == "SKY")
    return "o2abdlib, skybin"
}

def generateStage(server, myYaml, nexusFile) {
    return { 
        node {
            stage ('Prepare variables') {
                
                // select BBs which will be installed
                BBs_TO_DEPLOY = getBBsToDeploy(params.projectName)
                println("BB: ${BBs_TO_DEPLOY}")
                // get values from server.yaml for selected server
                hostValues = myYaml.get(params.OP_ENVIRONMENT_TYPE).get(params.OP_ENVIRONMENT).get(params.projectName).get(server)
                println("hostValues ${hostValues}")

                sh """
                    # Create script that exports environment variables for install.sh
                    echo "export modules="${BBs_TO_DEPLOY}"" > Deploy_parameters.sh
                    echo "export module="${BBs_TO_DEPLOY}"" >> Deploy_parameters.sh

                    if [ "${params.projectName}" == "SKY" ]
                    then 
                        echo This is the SKY branch 
                        echo "export rt_environment='${params.ENVIRONMENT_NAME}'" >> Deploy_parameters.sh
                    else
                        echo This is the OGW and O2A branch
                    fi

                    ssh -t -t -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key -l ${hostValues.account.first()} ${hostValues.host.first()} 'bash -s <<ENDSSH
                    echo Welcome to remote host: `hostname`
                    ls -la
ENDSSH' || exit 1
                    """
                // create name and url to dowload from nexus
                
            }
            if (params.OP_ENVIRONMENT == 'ACC3') {
                if (params.stage.contains('unzip')){
                    stage ('Unzip on server: $params.OP_ENVIRONMENT') {               
                        sh """          
                            echo Remove all old files in build area 
                            ssh -t -t -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key ${hostValues.host.first()} -l ${hostValues.account.first()} 'bash -s << ENDSSH
                                cd ${hostValues.homeDir.first()}/builds; 
                                rm -rf ${hostValues.homeDir.first()}/builds/*;
                                ls -la
ENDSSH' || exit 1
"""                         
                        sh "echo ${hostValues.account.first()}@${hostValues.host.first()}:${hostValues.homeDir.first()}"
                        sh """
                            echo Copy Deploy_parameters.sh to target server
                            
                            scp -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key Deploy_parameters.sh ${hostValues.account.first()}@${hostValues.host.first()}:${hostValues.homeDir.first()}/builds || exit 1
                            
                            echo Copy zip file to to target server 
                            scp -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key ${nexusFile} ${hostValues.account.first()}@${hostValues.host.first()}:${hostValues.homeDir.first()}/builds  || exit 1

                            ssh -t -t -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key ${hostValues.host.first()} -l ${hostValues.account.first()} 'bash -s << ENDSSH
                                set -x;
                                echo Unzip files;
                                cd ${hostValues.homeDir.first()}/builds; 
                                pwd; 
                                unzip ./${nexusFile}; 
                                chmod 755 ./install.sh ./Deploy_parameters.sh;

                                echo Unzip also o2abin;
                                cd ${hostValues.homeDir.first()}/builds; 
                                pwd; 
                                unzip ./o2abin4000.zip; 
                                ls -la ${hostValues.homeDir.first()}/builds/

                                echo Move o2abin/bin to ~/bin_future for comparison;
                                cd ${hostValues.homeDir.first()}; 
                                pwd; 
                                mv ${hostValues.homeDir.first()}/builds/o2abin4000/bin ${hostValues.homeDir.first()}/bin_future
                                ls -la ${hostValues.homeDir.first()}/bin_future/
ENDSSH' || exit 1
                        """

                    }
                }
                if (params.stage.contains('install')) {
                    stage('Install on server: $params.OP_ENVIRONMENT') {
                        sh """
                            if [ "${ProjectName}" == "SKY" ]
                            then 
                                echo Do nothing right now
                            else
                                if [[ "$ENVIRONMENT_ROOT" == "ACC" ]] 
                                then
                                echo "#########################################"
                                    echo Installing O2A or OGW in $ENVIRONMENT_NAME
                                    ssh -t -t -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key ${hostValues.host.first()} -l ${hostValues.account.first()} 'bash -s << ENDSSH
                                        whoami;
                                        . ./.profile; 
                                        echo Runnng ${hostValues.homeDir.first()}/bin/adminEnv_${ENVIRONMENT_NAME}.sh; 
                                        . ${hostValues.homeDir.first()}/bin/adminEnv_${ENVIRONMENT_NAME}.sh; 
                                        cd ${hostValues.homeDir.first()}/builds;  
                                        echo Wir haben dieses Home $HOME; 
                                        . ./Deploy_parameters.sh; 
                                        echo Now running install.sh; 
                                        ./install.sh  |tee |grep -v "inflated: "|grep -v "created: " > install.log
                                    ENDSSH'
                                    # ralph ssh -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key $INT_DEPLOY_HOST -l $INT_DEPLOY_ACCOUNT ksh -c "whoami; . ./.profile; echo Runnng  ${hostValues.homeDir.first()}/bin/adminEnv_${ENVIRONMENT_NAME}.sh; .  ${hostValues.homeDir.first()}/bin/adminEnv_${ENVIRONMENT_NAME}.sh ; cd  ${hostValues.homeDir.first()}/builds;  echo Wir haben dieses Home $HOME; . ./Deploy_parameters.sh; echo Now running install.sh; ./install.sh  |tee |grep -v "inflated: "|grep -v "created: " > install.log"  || exit 1
                                elif [[ "$ENVIRONMENT_ROOT" == "PROD" ]]
                                then 
                                    echo Not installing O2A or OGW in PROD yet
                                fi
                                echo Finished build_execute
                        
                                echo "[ERRORS]"
                                ssh -t -t -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key ${hostValues.host.first()} -l ${hostValues.account.first()} 'bash -s << ENDSSH
                                    . ./.profile; 
                                    cd  ${hostValues.homeDir.first()}/builds;  
                                    grep -iE 'error|warning' install.log 
                                ENDSSH'
                                #ssh -o StrictHostKeyChecking=no -i ogw_o2a_ssh_key $INT_DEPLOY_HOST -l $INT_DEPLOY_ACCOUNT ksh -c ". ./.profile; cd  ${hostValues.homeDir.first()}/builds;  grep -iE 'error|warning' install.log "
                            
                            fi
                            echo "Installing on ${server}."
                        """
                    
                    }
                }
            } else {
                        echo "Setup ${params.OP_ENVIRONMENT_TYPE} environment"
                        // sh "export HOME=${hostValues.homeDir.first()}; cd ~; echo $PATH"
                        echo "Running build execute script with parameters"
                        def confirmDialog = "Deploy project ${params.projectName} on ${params.OP_ENVIRONMENT} and server ${hostValues.host.first()} using wave: ${params.wave},  build: ${params.build}  and user ${hostValues.account.first()}"
                        releaseApprover = input message: confirmDialog,
                        submitterParameter: 'releaseApprover'
                        echo "${releaseApprover} is releasing ${params.projectName}"
                    }    
        }
    }
 }

node {
    NEXUS_REPOSITORY = "AMDOCS_RAW"
    // GIT_REPOSITORY = "deployvip.internal.vodafone.com:8080/bitbucket/scm/osf/amdocs.git"
    // GIT_CREDENTIAL = "jenkins_bitbucket"
    def myRepo = checkout scm

    
    // def INT_DEPLOY_HOST = ""
    // def INT_DEPLOY_ACCOUNT = ""
    def servers
    def nexusFile
    def nexusPath
    def BBs_TO_DEPLOY
    def hostValues
    def deliveryPath = ""
    //http://192.168.5.15:32712/service/rest/repository/browse/
    def nexusUrlTest = "http://192.168.5.15:32712/service/rest/repository/browse/"
    def repositoryTest = "AMDOCS_RAW"
    def jenkinsCredentialTest = "jenkins_nexus"
    def nexusUrl = "\$nexusUrl"
    def repository = "\$repository"
    def projectName = "\$projectName"
    def wave = "\$wave"
    def cred = [username:"\$cred.username",password:"\$cred.password"]
    def myYaml = readYaml file: "${env.WORKSPACE}/servers.yaml"
    if (params.projectName == "SKY") {
        nexusFile="${params.projectName}-V${params.wave}-${params.build}.zip"
        nexusPath="http://192.168.5.15:32712/repository/${NEXUS_REPOSITORY}/${params.projectName}/V${params.wave}/"
    } else {
        nexusFile="${params.projectName}-${params.wave}-${params.build}.zip"
        nexusPath="http://192.168.5.15:32712/repository/${NEXUS_REPOSITORY}/${params.projectName}/${params.wave}/"
    }
    println ("NexusFile: $nexusFile")
    println ("NexusPath: $nexusPath")


    properties([
                        parameters([
                            [   $class: 'ChoiceParameter', 
                                choiceType: 'PT_SINGLE_SELECT', 
                                name: 'dryRun',
                                randomName: 'choice-parameter-562fgg435534',
                                description: 'Dry run of the Jenkins job. Print variables. Update parameters..',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [], 
                                        sandbox: true, 
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [], 
                                        sandbox: true,
                                        script: """return ['disable','enable']""".stripIndent()
                                    ]
                                ]
                            ],
                            [   $class: 'CascadeChoiceParameter',
                                choiceType: 'PT_MULTI_SELECT',
                                description: 'What stages you want to run? (Can be multiple selection)',
                                filterLength: 1,
                                filterable: false,
                                name: 'stage',
                                randomName: 'choice-parameter-qweqweqwesd323326',
                                referencedParameters: 'dryRun',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [],
                                        sandbox: true,
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [],
                                        sandbox: true,
                                        script: """if (dryRun.equals('disable')){
                                            return  ['unzip','install']
                                        } else {
                                            return ['disabled:disabled']
                                        }
                                        """.stripIndent()
                                    ]
                                ]
                            ],  
                            [   $class: 'CascadeChoiceParameter',
                                choiceType: 'PT_SINGLE_SELECT',
                                description: 'Do you want to diff with current deployment?',
                                filterLength: 1,
                                filterable: false,
                                name: 'diff',
                                randomName: 'choice-parameter-qweqweqwesd324321',
                                referencedParameters: 'stage',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [],
                                        sandbox: true,
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [],
                                        sandbox: true,
                                        script: """if (stage.equals('unzip')){
                                            return ['disable', 'enable']
                                        } else {
                                            return ['disabled:disabled']
                                        }
                                        """.stripIndent()      
                                    ]
                                ]
                            ],
                            [  $class: 'ChoiceParameter', 
                                choiceType: 'PT_SINGLE_SELECT', 
                                name: 'OP_ENVIRONMENT_TYPE',
                                randomName: 'choice-parameter-562fgg435535',
                                description: 'Which environment you want to deploy?',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [], 
                                        sandbox: true, 
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [], 
                                        sandbox: true,
                                        script: """return ["${myYaml.keySet().join('","')}"]""".stripIndent()
                                    ]
                                ]
                            ],
                            [   $class: 'CascadeChoiceParameter',
                                choiceType: 'PT_SINGLE_SELECT',
                                description: 'Which sub environment you want to deploy?',
                                filterLength: 1,
                                filterable: true,
                                name: 'OP_ENVIRONMENT',
                                randomName: 'choice-parameter-qweqweqwesd124324',
                                referencedParameters: 'OP_ENVIRONMENT_TYPE',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [],
                                        sandbox: true,
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [],
                                        sandbox: true,
                                        script: """

                                        def venv = ${myYaml.inspect()}

                                        if(OP_ENVIRONMENT_TYPE?.trim()){
                                            return venv.get(OP_ENVIRONMENT_TYPE).keySet() as ArrayList
                                        } else {
                                            return []
                                        }""".stripIndent()
                                    ]
                                ]
                            ], 
                            [   $class: 'CascadeChoiceParameter',
                                choiceType: 'PT_SINGLE_SELECT',
                                description: 'Select the project to deploy',
                                filterLength: 1,
                                filterable: true,
                                name: 'projectName',
                                randomName: 'choice-parameter-qweqweqwesd324326',
                                referencedParameters: 'OP_ENVIRONMENT,OP_ENVIRONMENT_TYPE,allServers',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [],
                                        sandbox: true,
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [],
                                        sandbox: true,
                                        script: """def venv = ${myYaml.inspect()}
if(OP_ENVIRONMENT?.trim()){
    return venv.get(OP_ENVIRONMENT_TYPE).get(OP_ENVIRONMENT).keySet() as ArrayList
} else {
        return []
}""".stripIndent()
                                    ]
                                ]
                            ],
                            [   $class: 'CascadeChoiceParameter',
                                choiceType: 'PT_MULTI_SELECT',
                                description: 'Which server you want to use for deployment?',
                                filterLength: 1,
                                filterable: true,
                                name: 'serverName',
                                randomName: 'choice-parameter-qweqweqwesd324324',
                                referencedParameters: 'OP_ENVIRONMENT_TYPE,OP_ENVIRONMENT,projectName,allServers',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [],
                                        sandbox: true,
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [],
                                        sandbox: true,
                                        script: """if (allServers != 'all') {
                                        def venv = ${myYaml.inspect()}

                                        if(OP_ENVIRONMENT?.trim()){
                                            return venv.get(OP_ENVIRONMENT_TYPE).get(OP_ENVIRONMENT).get(projectName).keySet() as ArrayList
                                        } else {
                                            return []
                                        }
                                        } else {
                                            return ['disabled:disabled']
                                        }""".stripIndent()
                                    ]
                                ]
                            ], 
                            [   $class: 'ChoiceParameter', 
                                choiceType: 'PT_CHECKBOX', 
                                name: 'allServers',
                                randomName: 'choice-parameter-562fgg435543',
                                description: 'Deploy build on all servers in the environment',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [], 
                                        sandbox: true, 
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [], 
                                        sandbox: true,
                                        script: """return ['all']""".stripIndent()
                                    ]
                                ]
                            ],
                            [   $class: 'CascadeChoiceParameter',
                                choiceType: 'PT_SINGLE_SELECT',
                                description: 'Select needed wave for the deployment',
                                filterLength: 1,
                                filterable: true,
                                name: 'wave',
                                randomName: 'choice-parameter-qweqweqwesd324327',
                                referencedParameters: 'stage,projectName,dryRun',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [],
                                        sandbox: true,
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [],
                                        sandbox: true,
                                        script: """import org.jsoup.*
if (dryRun.equals('disable')){  
    def nexusUrl = 'http://192.168.5.15:32712/service/rest/repository/browse/'
    def repository = 'AMDOCS_RAW'
    def fullUrl  = '$nexusUrl$repository/$projectName/$wave/'
    def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
            com.cloudbees.plugins.credentials.Credentials.class,
            Jenkins.instance,
            null,
            null
    );
    def cred = jenkinsCredentials.find {it.id == 'jenkins_nexus'}
    def authString = '$cred.username:$cred.password'.getBytes().encodeBase64().toString()
    def doc = Jsoup
    .connect(fullUrl)
    .header('Authorization', 'Basic ' + authString)
    .get();
    def elements = doc.select('table > tbody > tr > td > a').text().findAll(~/\b(?!Parent|Directory)(\\w+)/);
    return elements.sort().reverse()
} else {
    return ['disabled:disabled']
}""".stripIndent()
                                    ]
                                ]
                            ], 
                            [   $class: 'CascadeChoiceParameter',
                                choiceType: 'PT_SINGLE_SELECT',
                                description: 'Select needed build number for the deployment',
                                filterLength: 1,
                                filterable: true,
                                name: 'build',
                                randomName: 'choice-parameter-qweqweqwesd324328',
                                referencedParameters: 'stage,wave,projectName,dryRun',
                                script: [
                                    $class: 'GroovyScript',
                                    fallbackScript: [
                                        classpath: [],
                                        sandbox: true,
                                        script: 'return ["error"]'
                                    ],
                                    script: [
                                        classpath: [],
                                        sandbox: false,
                                        script: """import org.jsoup.*
if (dryRun.equals('disable')){  
    def nexusUrl = 'http://192.168.5.15:32712/service/rest/repository/browse/'
    def repository = 'AMDOCS_RAW'
    def fullUrl  = '$nexusUrl$repository/$projectName/$wave/'
    def jenkinsCredentials = com.cloudbees.plugins.credentials.CredentialsProvider.lookupCredentials(
            com.cloudbees.plugins.credentials.Credentials.class,
            Jenkins.instance,
            null,
            null
    );
    def cred = jenkinsCredentials.find {it.id == 'jenkins_nexus'}
    def authString = '$cred.username:$cred.password'.getBytes().encodeBase64().toString()
    def doc = Jsoup
    .connect(fullUrl)
    .header('Authorization', 'Basic ' + authString)
    .get();
    def elements = doc.select('table > tbody > tr > td > a:contains($projectName-$wave)').text().findAll(~/\\d+(?=\.zip)/);
    return elements.sort().reverse()
} else {
    return ['disabled:disabled']
}""".stripIndent()     
                                    ]
                                ]
                            ]
                        ])
                    ])




    try {

      if (params.dryRun == 'enable'){
        stage("Dry Run") {
            println example('example', 'example')
            println getNexusRelease(nexusUrlTest, repositoryTest, jenkinsCredentialTest)
            
        
            // def myYaml = readYaml file: "${WORKSPACE}/servers.yaml"
            println("This is a Dry Run")
            def projects = myYaml.get(params.OP_ENVIRONMENT_TYPE).get(params.OP_ENVIRONMENT)
            println(projects)
            
            projects.each { entry ->
                echo "$entry.key"
                servers = myYaml.get(params.OP_ENVIRONMENT_TYPE).get(params.OP_ENVIRONMENT).get(entry.key)
                println(servers) 
                servers.each {server ->
                    echo "$server.key"
                    echo "$server.value"
                } 
            }
            currentBuild.result = "ABORTED" // DRY_RUN
        } 
    } 
    // else {
    //     echo "Skiped"
    //     // Utils.markStageSkippedForConditional('DryRun')
    // }
    
    stage("Download artifacts from Nexus") {
        if (params.stage.contains('unzip')){
            
            withCredentials([usernamePassword(credentialsId: "jenkins_nexus", passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USER')]) {
                println ("NexusFile: $nexusFile")
                println ("NexusPath: $nexusPath")

                sh """
                    echo Download master zip file ${nexusFile} from Nexus
                    echo ${NEXUS_USER}:${NEXUS_PASSWORD} ${nexusPath}${nexusFile}
                    curl --insecure -u ${NEXUS_USER}:${NEXUS_PASSWORD} ${nexusPath}${nexusFile} -o ${nexusFile} || exit 1
                    ls -la
                """
            }
        }
    }
    stage("Deploy components") {
        // hashicorp Part 1 - definition of hashicorp vault variable
        // def configuration = [vaultCredentialId: 'OSF-vault-credential', skipSslVerification: true]
        // def secrets = [[path: 'kv-jenkins-pipeline-OSF/ogw_o2a_private_key', secretValues: [[envVar: 'SSH_KEY', vaultKey: 'ssh_key']] ]]

        // hashicorp Part 2 - Retrieve key from  hashicorp vault and save key in temporary key file in workspace
        // withVault([configuration: configuration, vaultSecrets: secrets]) {
            // writeFile file: "ogw_o2a_ssh_key", text: env.SSH_KEY
            // sh 'chmod 600 ./ogw_o2a_ssh_key'
        // }
        // if (params.allServers == 'all'){
        //     servers =  myYaml.get(params.OP_ENVIRONMENT_TYPE).get(params.OP_ENVIRONMENT).get(params.projectName).keySet() as ArrayList    
        // } else {
        //     servers = params.serverName.split(',') // TODO check if working
        // }

        // def parallelStagesMap = servers.collectEntries {
        //                         ["${it}" : generateStage(it, myYaml, nexusFile)]
        //                     }
        // parallel parallelStagesMap 
    }
                
        println "This will run only if successful"
    
} catch (e) {
        echo 'This will run only if failed'

        // Since we're catching the exception in order to report on it,
        // we need to re-throw it, to ensure that the build is marked as failed
        throw e
    } finally {
        def currentResult = currentBuild.result ?: 'SUCCESS'
        if (currentResult == 'UNSTABLE') {
            echo 'This will run only if the run was marked as unstable'
        }

        def previousResult = currentBuild.previousBuild?.result
        if (previousResult != null && previousResult != currentResult) {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }

        echo 'This will always run'
        //TODO check if need to be rewrite to Groovy
        // hashicorp Part 3 - Remove temporary key file from workspace
        sh """
            if [ -f "./ogw_o2a_ssh_key" ]; then 
                rm ./ogw_o2a_ssh_key
            fi
        """ 
    }
}
