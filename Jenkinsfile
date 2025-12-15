pipeline {
    agent any

    environment {
        MKDOCS_IMAGE         = "squidfunk/mkdocs-material:9.7"
        DEPLOY_LOCATION      = "/net/awpbuild/tftpboot/awplus-gui/cloud/docs/mkdocs/site"
        IMAGE_DOCS_LOCATION      = "/net/awpbuild/tftpboot/awplus-gui/cloud/docs/images"
    }

    stages {
        stage('Copy in images docs') {
            steps {
                sh '''
                    for dir in ${IMAGE_DOCS_LOCATION}/*/; do
                        basename_dir=$(basename "$dir")
                        if [ -d "${dir}latest/docs" ]; then
                            if [ -n "$(find "${dir}latest/docs" -type f 2>/dev/null | head -1)" ]; then
                                mkdir -p "docs/images/${basename_dir}"
                                cp -r "${dir}latest/"docs/* "docs/images/${basename_dir}/"
                            fi
                        fi
                    done
                '''
            }
        }

        stage('Copy in images schema') {
            steps {
                sh '''
                    echo "Starting Copy in images schema stage"
                    echo "IMAGE_DOCS_LOCATION: ${IMAGE_DOCS_LOCATION}"
                    mkdir -p "docs/schema"
                    > docs/schema/SUMMARY.md
                    for dir in ${IMAGE_DOCS_LOCATION}/*/; do
                        echo "Processing directory: $dir"
                        basename_dir=$(basename "$dir")
                        echo "  Basename: $basename_dir"
                        if [ -d "${dir}latest/schema" ]; then
                            echo "  Found latest/schema folder"
                            if [ -n "$(find "${dir}latest/schema" -name '*.html' -type f 2>/dev/null | head -1)" ]; then
                                echo "  Folder contains HTML files - copying"
                                mkdir -p "docs/schema/${basename_dir}"
                                cp "${dir}latest/schema/"*.html "docs/schema/${basename_dir}/"
                                echo "  Copied to docs/schema/${basename_dir}/"
                                name=$(echo "$basename_dir" | tr '-' ' ')
                                echo "* [$name]($basename_dir/)" >> docs/schema/SUMMARY.md
                                echo "  Added to SUMMARY.md: * [$name]($basename_dir/)"
                            else
                                echo "  No HTML files found - skipping"
                            fi
                        else
                            echo "  No latest/schema folder - skipping"
                        fi
                    done
                    sort -o docs/schema/SUMMARY.md docs/schema/SUMMARY.md
                    echo "Copy in images schema stage complete"
                    echo "Final SUMMARY.md contents:"
                    cat docs/schema/SUMMARY.md
                '''
            }
        }

        stage('Create image docs summary') {
            steps {
                sh '''
                    > docs/images/SUMMARY.md
                    for dir in docs/images/*/; do
                        basename=$(basename "$dir")
                        name=$(echo "$basename" | tr '-' ' ')
                        echo "* [$name]($basename/)" >> docs/images/SUMMARY.md
                    done
                    sort -o docs/images/SUMMARY.md docs/images/SUMMARY.md
                    cat docs/images/SUMMARY.md
                    ls -la docs/images/
                '''
            }
        }

        stage('Create image schema summary') {
            steps {
                sh '''
                    > docs/schema/SUMMARY.md
                    for dir in docs/schema/*/; do
                        basename=$(basename "$dir")
                        name=$(echo "$basename" | tr '-' ' ')
                        echo "* [$name]($basename/)" >> docs/schema/SUMMARY.md
                    done
                    sort -o docs/images/SUMMARY.md docs/schema/SUMMARY.md
                    cat docs/schema/SUMMARY.md
                    ls -la docs/schema/
                '''
            }
        }

        stage('Build MkDocs site') {
            steps {
                sh """
                    docker run --rm \
                        -v "\$PWD":/docs \
                        --entrypoint sh \
                        ${MKDOCS_IMAGE} -c "pip install mkdocs-literate-nav && mkdocs build"
                """
            }
        }

        stage('Deploy MkDocs site') {
            steps {
                sh """
                    mkdir -p "${DEPLOY_LOCATION}"
                    rsync -av --delete site/ "${DEPLOY_LOCATION}"/
                """
            }
        }


    }
    post {
         always {
           cleanWs()
         }
    }
}