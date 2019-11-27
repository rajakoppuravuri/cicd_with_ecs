pipeline
{
agent any

parameters
{
  string defaultValue: '0.0.1-SNAPSHOT', description: 'Provide the image tag version', name: 'image_tag_version', trim: true
  string defaultValue: 'default', description: 'Provide the ecs cluster', name: 'ecs_cluster', trim: true
  string description: 'Provide the ecs cluster', name: 'SERVICE_NAME', trim: true
  string description: 'Provide the task family', name: 'TASK_FAMILY', trim: true
  string description: 'Provide the task count', name: 'DESIRED_COUNT', trim: true

  
  
  
}
environment
{
    registry = "662738233441.dkr.ecr.us-east-2.amazonaws.com"
    registry_url = "https://662738233441.dkr.ecr.us-east-2.amazonaws.com"
    image_repo = "cicd_poc"
    registryCredential = 'ecr:us-east-2:ecr_cred'
}

options {
  buildDiscarder logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '10', daysToKeepStr: '', numToKeepStr: '10')

}


stages
{
    stage ("Checkout")
    {
        steps
        {
            git branch:'master', url:'https://github.com/RajaKoppuravuri/cicd_with_ecs.git', credentialsId:'GitHub'
        }
    }

	stage ("Validate the parameters")
	{
	steps
		{
		  sh '''
          valid_params=
          echo "Validating the tag version::"
          image_tag_list=`aws ecr list-images --repository-name cicd_poc --region us-east-2 | jq '.imageIds' | jq 'values[].imageTag'|sed s/\\"//g`
          tag_array=($image_tag_list)
          for i in ${tag_array[@]}
          do
          
            if [[ "$i" == ${image_tag_version} ]]; then
              valid_params="yes"
              break
            fi
          done    

          if [[ ${valid_params} == "no" ]]; then
            echo "Please validate your inputs"
            #currentBuild.result = 'FAILED'
            #return
            exit 1
          fi  
        '''
		}
	}


    stage ("update task definition")
    {
        steps
        {
            script
            {

              sh ' sed -e "s;%tag_version%;${image_tag_version};g" ecs_task_definition.json>ecs_task_definition_${image_tag_version}.json '
            }
        }
    }

    stage ("register task definition")
    {
        steps
        {
            script
            {
              sh ''' aws ecs register-task-definition --family cicd_test --cli-input-json file://${WORKSPACE}/ecs_task_definition_${image_tag_version}.json --region us-east-2 '''
            }

        }
    }
    stage ("Update Service")
    {
        steps
        {
            script
            {
                sh '''
                    TASK_REVISION=`aws ecs describe-task-definition --task-definition cicd_test --region us-east-2|egrep "revision"| awk '{print $2}'`
                    
                    aws ecs update-service --cluster ${ecs_cluster} --service ${SERVICE_NAME} --region us-east-2 --task-definition ${TASK_FAMILY}:${TASK_REVISION} --desired-count ${DESIRED_COUNT}

                    '''
            }
        }
    }
}
post
{
	always
	{

		 cleanWs(
                cleanWhenAborted: false,
                cleanWhenFailure: false,
                cleanWhenNotBuilt: false,
                cleanWhenSuccess: false,
                cleanWhenUnstable: false,
                deleteDirs: false
            )
	}
}
}
