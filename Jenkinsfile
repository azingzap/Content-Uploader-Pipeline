pipeline {
    agent any

    parameters {
        choice(
            name: 'CATEGORY',
            choices: ['recipe', 'art', 'travel', 'technology'],
            description: 'Choose which category JSON to update'
        )
        string(name: 'TITLE', defaultValue: '', description: 'Card title (appears under image)')
        string(name: 'CAPTION', defaultValue: '', description: 'Short caption text')
        text(name: 'LARGE_CAPTION', defaultValue: '', description: 'Markdown text for large caption')
        string(name: 'IMAGE_PATH', defaultValue: './Photos/placeholder.jpg', description: 'Path to image file in the repo')
    }

    environment {
        WEBSITE_REPO = 'git@github.com:azingzap/MyWebsite.git'
        WEBSITE_BRANCH = 'main'
        GIT_CREDENTIALS = 'github-ssh-key'
        GIT_USER_NAME = 'Jenkins Bot'
        GIT_USER_EMAIL = 'jenkins@yourdomain.com'
    }

    stages {

        stage('Checkout Website Repository') {
            steps {
                echo "Cloning website repository from ${env.WEBSITE_REPO}"
                git branch: env.WEBSITE_BRANCH,
                    url: env.WEBSITE_REPO,
                    credentialsId: env.GIT_CREDENTIALS
            }
        }

        stage('Update JSON File') {
            steps {
                script {
                    def categoryMap = [
                        recipe: 'RecipeContent.json',
                        art: 'ArtContent.json',
                        travel: 'TravellingContent.json',
                        technology: 'TechnologyContent.json'
                    ]

                    def category = params.CATEGORY.toLowerCase()
                    if (!categoryMap.containsKey(category)) {
                        error "Unknown category: ${category}"
                    }

                    def filePath = "src/components/WebsiteContent/${categoryMap[category]}"
                    def jsonFile = []

                    echo "Updating JSON file: ${filePath}"

                    // Read JSON if exists
                    if (fileExists(filePath)) {
                        jsonFile = readJSON file: filePath
                    }

                    // Find max ID in current JSON
                    def maxId = jsonFile.collect { it.id.toInteger() }.max() ?: 0
                    def newId = (maxId + 1).toString()

                    // Create new entry
                    def newEntry = [
                        id: newId,
                        src: params.IMAGE_PATH,
                        alt: params.TITLE ?: "Image ${newId}",
                        caption: params.CAPTION ?: "",
                        largeCaption: params.LARGE_CAPTION ?: ""
                    ]

                    jsonFile << newEntry

                    // Write back the updated JSON
                    writeJSON file: filePath, json: jsonFile, pretty: 4
                    echo "Successfully added new entry to ${filePath}"
                    sh "cat '${filePath}'"
                }
            }
        }

        stage('Commit and Push Changes') {
            steps {
                sh '''
                    git config user.name "${GIT_USER_NAME}"
                    git config user.email "${GIT_USER_EMAIL}"
                    git add .
                    git commit -m "Auto-added ${CATEGORY} entry: ${TITLE}" || echo "No changes to commit"
                    git push origin ${WEBSITE_BRANCH}
                '''
            }
        }

    }

    post {
        success {
            echo " The Pipeline has completed successfully and the content has been uploaded to the JSON."
        }
        failure {
            echo " The Pipeline has failed — check console output for details."
        }
    }
}
