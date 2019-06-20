# Brief Overview of Gitlab's CI/CD Capabilities 

*What is CI/CD?* 

Essentially, it's an automated workflow for your software products. Instead of manually linting/testing/building/deploying 
your code you can automate it, and Gitlab has excellent features that enable you to configure and monitor your pipeline easily.

Here's a picture depicting a general CI/CD workflow: ![CI/CD](https://x2eh426qj0n44vhb23fwroq1-wpengine.netdna-ssl.com/wp-content/uploads/2019/03/cicd-pipeline-introduction.png)

All the magic happens in the .gitlab-ci.yml file. I'll go through a simple pipeline that I wrote. 


<pre><code>
stages: 
    - build 
    - test 


variables:
    PIP_CACHE_DIR: "$CI_PROJECT_DIR/.cache/pip"


# If you want to also cache the installed packages, you have to install
# them in a virtualenv and cache it as well.


cache:
    paths:
        - .cache/pip
        - venv/
        
before_script:
    - python3 -V  # Print out python version for debugging
    - pip3 install virtualenv
    - virtualenv venv
    - source venv/bin/activate


Build_Job: 
    stage: build 
    script: 
        - echo "Build Stage - Linting" 
        - pip3 install autopep8
        - autopep8 *.py    # dry-run, only print
        - autopep8 -i *.py # replace content in any .py files in the directory 
    tags: 
        - test
  
Test_Job: 
    stage: test 
    script: 
        - echo "Test Stage - Run Unit Tests"
        - pip3 install pytest 
        - pytest #runs all test files
    tags: 
        - test 

</code></pre>

To actually *run* this, we need to set up a Gitlab Runner. I set up a simple shell executor, but for scalability 
I would suggest using a Docker executor. You can find the necessary install instructions [here](https://docs.gitlab.com/runner/install/)
and configuration instructions [here](https://docs.gitlab.com/ee/ci/runners/) and registering instructions [here](https://docs.gitlab.com/runner/register/)

