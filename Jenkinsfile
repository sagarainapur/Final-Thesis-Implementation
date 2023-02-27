pipeline{
    agent any
    
    tools {
    maven 'maven'
    }
    
    // This section contains environment variables which are available for use in the pipeline's stages.
    environment {
		PATH = "$PATH:/usr/share/maven/bin"
        	region = "us-east-1"
        	docker_repo_uri = "498747127127.dkr.ecr.us-east-1.amazonaws.com/dockerimages"
	    	docker_image = "498747127127.dkr.ecr.us-east-1.amazonaws.com/dockerimages:latest"
	    	
		// ECS environment variables
	    	
		//task_def_arn = "arn:aws:ecs:us-east-1:407730735276:task-definition/first-run-task-definition:10"
	    	task_def_arn = "arn:aws:ecs:us-east-1:498747127127:task-definition/vote:2"
        	cluster = "CICD_ECS"
        	exec_role_arn = "arn:aws:iam::498747127127:role/ecsTaskExecutionRole"
    }
    
    stages{
        
       stage('GetCode'){
            steps{
                git branch: 'main', url: 'https://github.com/sagarainapur/Final-Thesis-Implementation.git'

            }
       }
       

	    
	stage('Dockerize') {
	     steps {
        	   // Get SHA1 of current commit
		   script {
            		   commit_id = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        	   }
                  // Build the Docker image
                  sh '''
		        ls -lash
			
		  	cd vote/
			
			ls -lash
			
			chmod +x Dockerfile app.py requirements.txt
			
			ls -lash
			##cd /var/lib/jenkins/workspace/CICD_ECS@tmp/durable-280cb0c1/
			
			#chmod +x script.sh
			
			# Build the Docker image
        		#docker build -t ${docker_repo_uri}:${commit_id} .
			docker build -t ${docker_repo_uri}:latest .
			
		
		   ''' 
	      }
	}
	
		    
	stage('Push to ECR') {
	     steps {
                  // Push the Docker image to AWS ECR
                  sh '''
		        ls -lash
			
						
        		# Get Docker login credentials for ECR
        		aws ecr get-login --no-include-email --region ${region} | sh
			
        		# Push Docker image
        		# docker push ${docker_repo_uri}:${commit_id}
			docker push ${docker_repo_uri}:latest
			
        		# Clean up
        		# docker rmi -f ${docker_repo_uri}:${commit_id}
			docker rmi -f ${docker_repo_uri}:latest
			
			
			
		
		   ''' 
	      }
	}
	
	
		
			
	stage('Approval for deployemnt') {

	     steps {

		  sh '''
		  
		  	echo " Pipeline will proceed after approval"
			
			
		  '''
		  
		  
		  input('Do you want to proceed?')
		  
		  
	      }
	}
	
	    
	    
	stage('ECS Depolyment') {
		
	     steps {

		// Override image field in taskdef file - taskdef.json
        	sh "sed -i 's|{{image}}|${docker_repo_uri}:latest|' taskdef.json"
        	// Create a new task definition revision
        	sh "aws ecs register-task-definition --execution-role-arn ${exec_role_arn} --cli-input-json file://taskdef.json --region ${region}"
        	// Update service on Fargate
        	sh "aws ecs update-service --cluster ${cluster} --service vote --task-definition ${task_def_arn} --region ${region}"
	      }
	}
        
    }
}
