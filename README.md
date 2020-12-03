# Drupal 9 CI Runner
I am using this image for automated tests of my drupal 9 installs. It's used in gitlab's gitlab-ci.yml files as image for docker runners.

## Diff to my Drupal 8 CI runner
The diff to my https://github.com/feikede/drupal8-docker/tree/master/drupal8-ci-runner Drupal 8 CI runner is
* changed apache2 ports to non-privileged port 8080 and 8082
* changed php version from 7.2 to 7.3
* changed composer version from 1 to 2 
* changed apache2 version from 2.4.6 to 2.4.38
* increased php mem to 768 MB, upload size to 10 MB
* added sodium to php build
* added redis client to php build
* activated apache2 mod_rewrite in image

With this image you can run Drupal sites 8.4 up to at least 9.1


## Here's what I do for the phpunit test in my .gitlab-ci.yml

```yml
test_job:
  only:
    - dev
  stage: test
  script:
    - pwd
    - echo $CI_JOB_ID
    - echo $CI_PROJECT_NAME
    # setup testrun
    - cd /builds/mysecret
    - ls -al
    - export COMPOSER_ALLOW_SUPERUSER=1
    # tweak apache for drupal must-haves
    - sed -i 's:DocumentRoot /var/www/html:DocumentRoot /builds/mysecret/web:g' /etc/apache2/sites-enabled/000-default.conf
    - sed -i 's:/var/www/:/builds/mysecret/:g' /etc/apache2/apache2.conf
    - sed -i 's:AllowOverride None:AllowOverride All:g' /etc/apache2/apache2.conf
    # create drupal scaffoldings and lib stuff
    - composer install
    # cp phpunit test-setup to core
    - cp phpunit.citest.xml web/core/phpunit.xml
    - ps -eaf
    # add mod_rewrite for path_aliases
    - a2enmod rewrite
    # install test module and run it
    - /etc/init.d/apache2 start
    - ps -eaf
    - mkdir -p /builds/mysecret/web/sites/simpletest/browser_output
    - chown -R www-data /builds/mysecret/web/sites
    - sudo -u www-data -E vendor/bin/phpunit --configuration web/core web/modules/custom/testautomat/tests/src/Functional/FullTest.php
    - php -r 'echo "\nFantastico\n";'
    - ls -altr /builds/mysecret/web/sites/simpletest/browser_output
    # echo last test-output
    - cat `ls -1tr /builds/mysecret/web/sites/simpletest/browser_output/*html | tail -1`

```


Have fun.
