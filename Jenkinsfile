      
node {
    def address = "github.com/mo40070369/SamplePOC.git"

    def  notify_users = ""
    def newCommitComments= '';
    def project_name = "";
    def commit_id_exist = false;
    def splunkurl = ""
    def ei_service_url=""
    def earth_service_check_url=""
    def earth_service_info_url=""
    def ei_service_related_info_html=""
    def isRunByAdmin = env.Base_EI_Project?true:false
    def build_email_title_suffix = ""    
   

try {
	  
    stage("Clear workspace"){ 
        powershell 'rm -r -fo ./*'
    }
	
    def ei_service_project_tem_path='ei_service_project_tem_path';
    def base_ei_project_tem_path='base_ei_project_tem_path';

	def commitComments = '';
    def gitURL = '';
    def gitProjectName = '';
    def commit_id = '';
	def is_car_build_error = false;
    def is_car_deploy_error = false;
	
    stage("Checkout code: current project"){
        bat "mkdir ${ei_service_project_tem_path}"
        dir("${ei_service_project_tem_path}"){
            if(env.EI_Service_Project){
                EI_Service_Project = env.EI_Service_Project.replaceAll("https?://","")
               git credentialsId: 'devops_svc', url: "http://${EI_Service_Project}", branch: "${env.BRANCH_NAME}"
                //   git credentialsId: 'devops_svc', url: "http://${EI_Service_Project}", branch: "dev"

            }else{
               checkout scm 
            }
            // gitURL = sh(returnStdout: true, script: 'git remote -v|grep -v push|grep -oE http.*git|cat')
            gitURL = bat(returnStdout: true, script: 'git remote -v| find "http*.git" /v| find "push"')
         // gitProjectName  = sh(returnStdout: true, script: "git remote -v|grep -v push|grep -oE http.*git|cat|sed 's%^.*lenovo.com/%%g'|sed 's/.git\$//g'")
            gitProjectName  = powershell (returnStdout: true, script: "Set-Alias -Name sed -Value C:/'Program Files'/Git/usr/bin/sed.exe;git remote -v|findstr /R 'push'|findstr 'http.*git'|sed 's%^.*lenovo.com/%%g'|sed 's/.git\$//g'")
            //  gitProjectName  = bat(returnStdout: true, script: 'git remote -v|  findstr /R "push"')
            build_email_title_suffix = "[(${env.BRANCH_NAME})${gitProjectName}] "
            echo "build_email_title_suffix is $build_email_title_suffix"
            commit_id = bat(returnStdout: true, script: 'git rev-parse HEAD')
            commitComments = bat(returnStdout: true, script: 'git log -1 ')
			newCommitComments= "Add new Car to ${env.BRANCH_NAME} ${gitProjectName}\nBranch:${env.BRANCH_NAME}\nCommit Comments:\n${commitComments}";
            echo "newCommitComments ${newCommitComments}"
        }
    }
 
     def props = readYaml file: "${ei_service_project_tem_path}/jenkinsfile.pipeline.yml"
     echo "Base_EI_Project = ${env.Base_EI_Project}"
     def  Base_EI_Project = getNotNullValue(env.Base_EI_Project,props.base_mi_project)
	 echo "Base_EI_Project = ${Base_EI_Project}"
	      Base_EI_Project = getNotNullValue(Base_EI_Project,address);
          Base_EI_Project = Base_EI_Project.replaceAll("https?://","")
	 echo "Base_EI_Project = ${Base_EI_Project}"
     def  is_vm_target_system = "${Base_EI_Project}".contains("shipment_br.git"); 
	 def is_need_build = env.BRANCH_NAME != 'master' || isRunByAdmin || is_vm_target_system ;
	 notify_users = concatNotifyUsers(props.notify_users,env.Notify_Users);

	 if(!is_need_build){
	 
	     currentBuild.setDescription("Skip build :RANCH_NAME: ${env.BRANCH_NAME} , Base_EI_Project: ${Base_EI_Project}");
	 
	 }else{
     
	stage("Checkout code: configuration"){
        dir(base_ei_project_tem_path){

try{
           git credentialsId: 'devops_svc', url: "http://${Base_EI_Project}", branch: "${env.BRANCH_NAME}"
}catch(err){
     stage('Perpare Base EI') { 
        echo "Trying to create a Base EI Branch With dev"
        build job: 'integration/Integration Foundation/EI_MTP_Automation/MTP-Tools/Copy-Base-EI-Branch',parameters: [string(name: 'Base_EI_Project', value: Base_EI_Project),string(name: 'Source_Branch', value: 'dev'),string(name: 'Target_Branch', value: env.BRANCH_NAME) ], wait: true
    }
         git credentialsId: 'devops_svc', url: "http://${Base_EI_Project}", branch: "${env.BRANCH_NAME}"
}
            def props1 = readYaml file: "jenkinsfile.pipeline.yml"
            splunkurl = props1.splunkurl
            ei_service_url=props1.ei_service_url
            earth_service_check_url=props1.earth_service_check_url
            earth_service_info_url=props1.earth_service_info_url
            ei_service_related_info_html=" \n\nYour EI Service: ${ei_service_url}\n (Test Env can add a header skip-verify equaling true to skip verify)<text/>  \n\nBase EI Project: http://${Base_EI_Project} \n\nBase EI Project Logs: <a href='${splunkurl}'>splunkurl</a>  \n\n  Earth Service: ${earth_service_info_url} \n\nYour Commit Info: \n<text>${newCommitComments}</text>\n\nBuild Detail: ${env.BUILD_URL}console"
        
        }  
    }

    stage('Replace EI Serive Parameters') {
        dir ("${ei_service_project_tem_path}") {
            
			if(is_vm_target_system) {
                 echo 'Target is VM ESB , Changing Role EnterpriseIntegrator to  EnterpriseServiceBus ... '
                //  bat ''' find -name artifact.xml |forfiles sed -i 's/serverRole="EnterpriseIntegrator"/serverRole="EnterpriseServiceBus"/g' '''
                //  bat '''echo "check artifact.xml "  && find -name artifact.xml |forfiles type '''
             }
            bat 'dir'
            
              dir ("WMSLite_OPS_Shipment_BR"){
                bat 'dir'
                // bat 'ls -ltr'
                props.environments.main.each {
                updateProperty("{${it.key}}", "${it.value}" ,'*/src/main/synapse-config/*/*.xml')
                updateProperty("${it.key}", "${it.value}" ,'*/dataservice/*.dbs')
            }
              }
               
            
        }
    }
        def build_result = ""
        def deploy_result = ""
        stage("Build in Maven"){
            echo "cd ${ei_service_project_tem_path}"
            echo "cd ${ei_service_project_tem_path}/Sample && mvn clean install  -Dmaven.test.skip=true"
          build_result = bat (returnStdout: true, script: "cd ${ei_service_project_tem_path}/Sample && mvn clean install -Dmaven.test.skip=true || echo success")
          is_car_build_error = build_result.contains("[ERROR]")
        }
        
        if(is_car_build_error){
        currentBuild.result = 'FAILURE'
        currentBuild.setDescription("${build_email_title_suffix} EI Car Build Error")
		    emailext (
            subject: "${build_email_title_suffix} EI Car Build Error", 
            mimetype: 'text/html', 
            to: notify_users,
            body: build_result
            ) 
		}else{

        // stage("Cope car"){
        //     echo "print base mi proj"
        //     echo "${base_ei_project_tem_path}"
        //     // bat "copy ${ei_service_project_tem_path}/*_capp/target/*.car ${ei_deploy}"
        // }
        stage("Deploy car in server"){
            echo "deploy car in server"
            deploy_result = bat (returnStdout: true, script: "cd ${ei_service_project_tem_path}/Sample && mvn  deploy -Dmaven.deploy.skip=true -Dmaven.car.deploy.skip=false -Dmaven.test.skip=true -Dmaven.wagon.http.ssl.insecure=true || echo success")
             is_car_deploy_error = deploy_result.contains("[ERROR]")

        }
        if(is_car_deploy_error){
        currentBuild.result = 'FAILURE'
        currentBuild.setDescription("${build_email_title_suffix} EI Car Deploy Error")
		    emailext (
            subject: "${build_email_title_suffix} EI Car Deploy Error", 
            mimetype: 'text/html', 
            to: notify_users,
            body: deploy_result
            ) 
		}
        
        stage("Record Commit ID"){
              def template_path= "${base_ei_project_tem_path}/repository/deployment/server/synapse-configs/default/templates/"

              if(fileExists("${template_path}EI_Service_CommitID_Recorder.xml")){
                  commit_id_exist = true;
                  def commit_id_list_string = bat (returnStdout: true, script: "type ${template_path}EI_Service_CommitID_Recorder.xml|FIND -oE 'commit_id_start.*commit_id_end'")
                  def new_commit_id_list_string = "";
                  def commit_id_list = commit_id_list_string.split(',');
                  commit_id_list.eachWithIndex { item, i ->
                     if(i==0){
                        new_commit_id_list_string = item.trim()+","+commit_id.trim();
                     }else
                     if(i<20){
                          new_commit_id_list_string+=","+item.trim();
                     }
                  }
                  new_commit_id_list_string+=",commit_id_end";

                
                  echo "commit_id_list_string=${commit_id_list_string}"
                  echo "new_commit_id_list_string=${new_commit_id_list_string}"
                  def sed_command = "sed -i 's|${commit_id_list_string.trim()}|${new_commit_id_list_string.trim()}|g' ${template_path}EI_Service_CommitID_Recorder.xml"
                  echo "sed_command=${sed_command}"
                  bat "${sed_command}"
              }
        }        
       		}         
    // }
    
   if(!is_car_build_error){
    // stage("commit & push code to Base_EI_Project"){
    //     dir(base_ei_project_tem_path){
    //          withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'devops_svc',usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]){
    //             powershell 'git config --global user.email "wso2devops@lenovo.com"'
    //             powershell 'git config --global user.name "wso2devops"'
    //             powershell 'git add --all'
    //             powershell "git commit -m '${newCommitComments}  \n Build Detail: ${env.BUILD_URL}console'"
    //             powershell "git push http://$USERNAME:$PASSWORD@${Base_EI_Project} ${env.BRANCH_NAME} || echo success"
    //             currentBuild.setDescription("Success deploy : ${newCommitComments}");
    //     }
    //     }
    // }


	echo "earth_service_check_url = ${earth_service_check_url} ,commit_id_exist = ${commit_id_exist}"
    if(commit_id_exist && earth_service_check_url ){
	    stage("Wait for Earth Deployment"){
        project_name = gitProjectName.split('/')[1];
        try {
            timeout(time: 180, unit: 'SECONDS') {
                sleep 20
				waitUntil {
				     earth_service_check_url = earth_service_check_url.replaceAll("service_commit_id",commit_id.trim())
                     echo "earth_service_check_url=${earth_service_check_url}"
                     def key_words = 'Commit ID Exist'
                     def earth_ei_service_response = bat(returnStdout: true, script: "${earth_service_check_url} || echo success")
                     echo "earth_ei_service_response=${earth_ei_service_response}"
					 if(earth_ei_service_response.contains(key_words)){
					  return true;
					 }
					 return false;
				}
                emailext (
                subject: "${build_email_title_suffix} EI Car Deploy Successfully", 
                mimetype: 'text/html', 
                to: notify_users,
                body: "You can access your service now. ${ei_service_related_info_html}"
             ) 
			}
            }catch (err) {
        currentBuild.result = 'FAILURE'
        currentBuild.setDescription("Wait for Earth Deployment Failed :RANCH_NAME: ${env.BRANCH_NAME} , Base_EI_Project: ${Base_EI_Project}")

            emailext (
            subject: "${build_email_title_suffix} EI Car Deploy Failedd", 
            mimetype: 'text/html', 
            to: notify_users,
            body: "${build_email_title_suffix} deploy failed , is car <a href='${splunkurl} Ignoring Carbon Application'> deployment error </a>?   \n\nError: ${err} \n\n ${ei_service_related_info_html}  \n\n if can not find the RC , please contact with Integration BASIS .\n\nEmail: -Integration_BASIS@lenovo.com \n\n Detail: ${env.BUILD_URL}console ",
             ) 
        
    }
         
            }
 }else{

    echo "notify_users = ${notify_users}"
    echo "${build_email_title_suffix} EI Car Deploy to Base EI Project Successfully "
    // echo "parseSplunkUrl ${parseSplunkUrl(Base_EI_Project)}"
    echo "Base_EI_Project ${Base_EI_Project}"
    echo "newCommitComments ${newCommitComments}"
    echo "BUILD_URL ${env.BUILD_URL}"

    echo "${build_email_title_suffix} EI Car Deploy to Base EI Project Successfully \n\nAll Environment Base EI Project Logs: <a href='${parseSplunkUrl(Base_EI_Project)}'>splunk</a> \n\nCheck If zzzzzzz_MI_Health_Check Car Deployed : <a href='${parseSplunkUrl(Base_EI_Project)} zzzzzzz_MI_Health_Check '>splunk</a> \n\nBase EI Project: http://${Base_EI_Project} \n \n\nYour Commit Info: \n<text>${newCommitComments}</text>\n\nBuild Detail: ${env.BUILD_URL}console \n \n In this Base EI Project version , We can not check whether the EI Service was deployed to Earth , and whether  there is other EI Service deployed failed make your EI Service unable to be deployed.\n\nif you need response your EI Service deployment status when you deploy ,  you can contact with Integration Basis to upgrade the Base EI Project to release/v1.1 .  \n Integration Basis: -Integration_BASIS@lenovo.com "

    
       emailext (
                subject: "${build_email_title_suffix} EI Car Deploy to Base EI Project Successfully ", 
                mimetype: 'text/html', 
                to: 'javidm@motorola.com',
                body: "${build_email_title_suffix} EI Car Deploy to Base EI Project Successfully \n\n All Environment Base EI Project Logs: <a href='${parseSplunkUrl(Base_EI_Project)}'>splunk</a> \n\nCheck If zzzzzzz_MI_Health_Check Car Deployed : <a href='${parseSplunkUrl(Base_EI_Project)} zzzzzzz_MI_Health_Check '>splunk</a> \n\nBase EI Project: http://${Base_EI_Project} \n \n\nYour Commit Info: \n<text>${newCommitComments}</text>\n\nBuild Detail: ${env.BUILD_URL}console \n \n In this Base EI Project version , We can not check whether the EI Service was deployed to Earth , and whether  there is other EI Service deployed failed make your EI Service unable to be deployed.\n\nif you need response your EI Service deployment status when you deploy ,  you can contact with Integration Basis to upgrade the Base EI Project to release/v1.1 .  \n Integration Basis: -Integration_BASIS@lenovo.com "
                // body: 'CAR has deployed successfully'
             ) 
    
 } }
    }
  } catch (err) {
        currentBuild.result = 'FAILURE'
        def to_users = notify_users;
        
        if(!to_users){
            to_users =  env.Notify_Users;
        }
        stage ('notify') {
            emailext to: "${to_users}",
            recipientProviders: [[$class: 'RequesterRecipientProvider'],[$class: 'DevelopersRecipientProvider']],
            subject: "${build_email_title_suffix} EI Car Deploy Failed",
            body: "${build_email_title_suffix} deploy failed \n\nError: ${err} \n\n${ei_service_related_info_html} \n\n if can not find the RC , please contact with Integration BASIS .\n\nEmail: -Integration_BASIS@lenovo.com ",
            mimeType: 'text/html'
        }
    }

}

def concatNotifyUsers(userList1,userList2){

        if(userList1 && userList2){
            return userList1+','+userList2;
        }
        
        if(userList1){
           return userList1;
        }

        if(userList2){
           return userList2;
        }

        return '-Integration_BASIS@lenovo.com';
}
def updateProperty(property, value, file) { 
     echo 'Inside updateProperty method..'
         echo "file name: $file + Property: $property  value: $value"

        escapedProperty = property.replace('[', '\\[').replace(']', '\\]').replace('.', '\\.')
        echo "file name: $file + escapedProperty: $escapedProperty  value: $value"      
      def updateStatus = powershell (returnStdout: true, script: "Set-Alias -Name sed -Value C:/'Program Files'/Git/usr/bin/sed.exe; sed -i 's|$escapedProperty|$value|g' $file | echo success")


// def stdout1 = powershell(returnStdout: true, script: """
// get-content $file """ + ' | %{$_ -replace '+ """ $escapedProperty,"http://100.67.10.91:9763/services/Moto_eCommerce_Delivered_Orders_DataService" }
// """

// )
//     println stdout1

   
        // echo "sed -i 's|$escapedProperty|$value|g' $file"
        
        
        //  powershell ("get-content */src/main/synapse-config/*/*.xml | %{$_ -replace 'WMSLITE_OPS_SHIPMENT_DSS_EP','http://100.67.10.91:9763/services/Moto_eCommerce_Delivered_Orders_DataService'}")
        
    //    def updateStatus = powershell (returnStdout: true, script: "get-content */src/main/synapse-config/*/WMSLite_OPS_Shipment_DSS_EP.xml | %{$_ -replace "WMSLITE_OPS_SHIPMENT_DSS_EP","http://100.67.10.91:9763/services/Moto_eCommerce_Delivered_Orders_DataService"})
    //   def updateStatus = powershell (returnStdout: true, script: "get-content $file | %{$_ -replace $escapedProperty,$value}")

       println updateStatus
}

def parseSplunkUrl(Base_EI_Project){
    echo  "********Enter*********"
    // def base_ei_project_name = Base_EI_Project.split('openesb-image-config/')[1].split('.git')[0]
    def base_ei_project_name = Base_EI_Project
    echo "base_ei_project_name ${base_ei_project_name}"
    def env = env.BRANCH_NAME == 'main'?'PROD':env.BRANCH_NAME;
   return " ${base_ei_project_name} AND ${env}";
}

def getNotNullValue(value, defauleValue) { 
      if(value){
          return value;
      }
      return defauleValue;
}

