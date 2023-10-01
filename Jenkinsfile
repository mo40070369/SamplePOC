// Define jobName here

def jobName = env.JOB_NAME

 

pipeline {

    agent any

    environment {

        inputdata = '' // Define inputdata at the pipeline level

        carbonAppName = 'SuccessSampleGuarantyDelivaryCompositeExporter'

        customJobName = "${jobName}"

    }

   

    stages {

        stage('Call Management API') { // A single stage that encompasses both steps

            steps {

                script {

                    try {

                        echo "Current Job Name: ${jobName}"


 

                    } catch (Exception e) {

                        // Handle the exception when there is an error in the stage

                        echo "An error occurred in the 'Call Management API' stage: ${e.getMessage()}"

                        // Optionally, you can take additional actions or set a build result here

                        currentBuild.result = 'FAILURE' // Set the build result to FAILURE

                        echo "currentBuildStatus1: ${currentBuild.result}"

                        // step3 to Check Current Build Status

                        echo "Current Job Name: ${jobName}"

                    def currentBuildStatus = currentBuild.result

                    echo "currentBuildStatus: ${currentBuildStatus}"

                    echo "currentBuildStatus2: ${currentBuildStatus}"

                    // if (currentBuildStatus == 'SUCCESS') {

                    //     echo "The current build was successful."

                    // } else {

                        echo "The current build was not successful."

 

                        def lastBuild = build(job: "${jobName}", propagate: false, wait: false)

                        //if (currentBuild.result < 'SUCCESS')  {

                            echo "${jobName}"

                           

                            def lastSuccessfulBuild = build(job: "${jobName}", propagate: false, wait: true, parameters: [[$class: 'RebuildSettings', rebuild: true]])

                             echo "The last successful build (Build #${lastSuccessfulBuild.number}) was successful."

                            // if (lastSuccessfulBuild.result > 'SUCCESS') {

                            //     echo "The last successful build (Build #${lastSuccessfulBuild.number}) was successful."

                            // } else {

                            //     error "No last successful build found for ${jobName}"

                            //}

                        //}

                //}

            }

        }

       

       

                    }

                }

            }

        }

   
