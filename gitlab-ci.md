# Brief Overview of Gitlab's CI/CD Capabilities 

*What is CI/CD?* 

Essentially, it's an automated workflow for your software products. Instead of manually linting/testing/building/deploying 
your code you can automate it, and Gitlab has excellent features that enable you to configure and monitor your pipeline easily.

Here's a picture depicting a general CI/CD workflow: ![CI/CD](https://x2eh426qj0n44vhb23fwroq1-wpengine.netdna-ssl.com/wp-content/uploads/2019/03/cicd-pipeline-introduction.png)

All the magic happens in the .gitlab-ci.yml file. I'll go through a simple pipeline that I wrote. 


<pre><code>
# -----Simple CI/CD Pipeline-----

# First, create a virtual environment and install your dependencies, 
# caching them so that each job retains them. 

# Next, lint all Python files contained within Pipelines/dags/ and Pipelines/dags/DATAFLOW directories 

# Then, run unit tests. 

# Finally, update the Apache Airflow job accordingly


stages: 
    - build 
    - test 
    - deploy 


variables:
    PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"


cache:
    paths:
        - .cache/pip
        - venv/
        
before_script:
    - python3 -V 
    - pip3 install virtualenv
    - virtualenv venv
    - source venv/bin/activate


Lint_Job: 
    stage: build 
    script: 
        - pip3 install autopep8
        - pip3 install pylint 
        - cd Pipelines/dags
        - autopep8 *.py    # dry-run, only print
        - autopep8 -i *.py # replace content
        - pylint *.py **/*.py #verify that files were properly linted 
    tags: 
        - test
  
Test_Job: 
    stage: test 
    script: 
        - echo "Test Stage - Run Unit Tests"
        - pip3 install pytest 
        - pytest # runs all test files in the form *_test.py or test_*.py 
    tags: 
        - test 

Update_Airflow_Job: 
    stage: deploy 
    script: 
        - echo "Deploy Stage - Update Airflow Job"
        - gsutil -m cp "$(< wheels.txt)" "$(< dag_wheels.txt)"
        - gsutil -m rsync -rd dags "$(< dag_location.txt)"
    tags: 
        - test 

</code></pre>

To actually *run* this, we need to set up a Gitlab Runner. I set up a simple shell executor, but for scalability 
I would suggest using a Docker executor. You can find the necessary install instructions [here](https://docs.gitlab.com/runner/install/)
and configuration instructions [here](https://docs.gitlab.com/ee/ci/runners/) and registering instructions [here](https://docs.gitlab.com/runner/register/)


Note: further complexities can be added such as customized trigger events and more robust integration testing 
