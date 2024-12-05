pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('s3-redshift')  // Jenkins reference to AWS Access Key ID
        AWS_SECRET_ACCESS_KEY = credentials('s3-redshift')  // Jenkins reference to AWS Secret Access Key
        S3_BUCKET = 's3://bucket_name'  // S3 path
        REDSHIFT_HOST = 'redshiftcluster.eu-north-1.redshift.amazonaws.com'  // Redshift endpoint
        REDSHIFT_DB = 'dev'  // Redshift database name
        REDSHIFT_USER = 'cse-a'  // Redshift username
        REDSHIFT_PASSWORD = 'CSE-a-123'  // Redshift password
        REDSHIFT_PORT = '5439'  // Redshift port (default 5439)
        TABLE_NAME = "sem1"  // Redshift table name
        IAM_ROLE_ARN = 'arn:aws:iam::acc_num:role/role_name'  // IAM role ARN for Redshift to access S3
    }

    stages {
        stage('Install Dependencies') {
            steps {
                script {
                    // Check if awscli is installed
                    def awsInstalled = sh(script: 'which aws', returnStatus: true) == 0

                    if (!awsInstalled) {
                        echo 'AWS CLI is not installed. Installing...'
                        sh 'sudo apt-get update'
                        sh 'sudo apt-get install -y awscli'
                    } else {
                        echo 'AWS CLI is already installed. Skipping installation.'
                    }
                }
            }
        }
		stage('Check and Install PostgreSQL Client') {
            steps {
                script {
                    // Check if PostgreSQL client is installed
                    def psqlInstalled = sh(script: 'which psql', returnStatus: true) == 0

                    if (!psqlInstalled) {
                        echo 'PostgreSQL Client is not installed. Installing...'
                        sh 'sudo apt-get install -y postgresql-client'
                    } else {
                        echo 'PostgreSQL Client is already installed. Skipping installation.'
                    }
                }
            }
        }

        stage('Download Data from S3') {
            steps {
                script {
                    // Use AWS CLI to download the CSV file from S3
                    sh "aws s3 cp ${S3_BUCKET} ./cse-a-students-sem-1-marks.csv"
                }
            }
        }

        stage('Load Data to Redshift') {
            steps {
                script {
                    // Redshift COPY command to load data into Redshift
                    def copySql = """
                    COPY ${TABLE_NAME}
                    FROM '${S3_BUCKET}'
                    IAM_ROLE '${IAM_ROLE_ARN}'
                    DELIMITER ','
                    IGNOREHEADER 1
                    CSV;
                    """
                    
                    // Use psql to run the COPY command in Redshift
                    sh """
                        PGPASSWORD=${REDSHIFT_PASSWORD} psql -h ${REDSHIFT_HOST} -p ${REDSHIFT_PORT} -U ${REDSHIFT_USER} -d ${REDSHIFT_DB} -c "${copySql}"
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Clean up the downloaded CSV file from the workspace
                    sh 'rm -f ./cse-a-students-sem-1-marks.csv'
                }
            }
        }
    }

    post {
        always {
            echo 'pipeline execution completed!!'
        }
        
    }
}
