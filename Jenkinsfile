pipeline {
    agent any

    environment {
        OPTION_A = "Cats"
        OPTION_B = "Dogs"
        PORT = "5000"  // Using port 5000
        VENV_DIR = ".venv"
    }

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/MoizArcana/voteapp.git'
            }
        }

        stage('Set up Python Environment') {
            steps {
                sh '''
                    # Create virtual environment
                    python3 -m venv ${VENV_DIR}

                    # Activate virtual environment (using dot instead of source)
                    . ${VENV_DIR}/bin/activate

                    # Upgrade pip and install requirements
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Application') {
            steps {
                sh '''
                    # Kill any existing app on port 5000 (optional, Unix only)
                    fuser -k ${PORT}/tcp || true

                    # Activate virtual environment (using dot instead of source)
                    . ${VENV_DIR}/bin/activate

                    # Run the Flask app with Gunicorn on port 5000
                    nohup gunicorn app:app -b 0.0.0.0:${PORT} \
                        --log-file - --access-logfile - --workers 4 --keep-alive 0 > app.log 2>&1 &

                    # Sleep for 60 seconds to keep the app running and allow checking
                    sleep 60
                '''
            }
        }
    }

    post {
        success {
            echo 'Flask app started successfully.'
        }
        failure {
            echo 'There was a problem running the app.'
        }
    }
}
