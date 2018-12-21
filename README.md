# Drupal 8 - Docker - GitLab CI
Example setup for a Drupal 8 environment, PHPCS, PhpUnit, Behat and
deployment with Docker and GitLab CI.

## Drupal 8
Basic composer.json file which installs Drupal 8 with PHPCS, PhpUnit and behat.
The composer.json has some scripts to help running the tests:

* **phpcs-all**: check all files based on Drupal ruleset (with exception of longer lines than 80 chars)
* **phpcs**: check changed (git diff) files based on Drupal ruleset (with exception of longer lines than 80 chars)
* **phpunit**: run phpunit on test files (see [config/phpunit.xml](config/phpunit.xml)).
* **phpunit_nocolors**: same as phpunit, but then output without colors. Used for CI.
* **behat**: run behat tests.
* **quick_tests**: run 'Quick tests' for CI. Separate from behat since that normally takes a while and we can run them in parallel.

## Dockerfile
Currently, I'm just using the Dockerfile to create an image where to run all the tests during the CI process.

The Dockerfile is just a simple PHP 7 image with some extra applications. Also composer is installed and xdebug configured
for getting code coverage.

NOTE: This Dockerfile shouldn't be used directly for production environments.

## GitLab CI
The docker image is only created when you run it with a branch starting with 'docker-', so you need to do that for the first time.
The image created will be stored in the registry of your repository.

I was using other private repositories for my Drupal environment, so I need to allow the runner to access them. To do that, 
add a variable to the CI/CD settings called 'SSH_KEY_COMPOSER' which holds the ssh key (see https://docs.gitlab.com/ee/ci/ssh_keys).

1. Build dev stage:
- Setup correct composer caching for the runner
- Run composer install
- Create an artifact for the next stage: test

2. Test stage (2 jobs):
* Coding standards + phpunit
  - Run composer's script 'quick_tests'
* Behat:
  - Setup a MariaDB environment
  - Symlink configuration files for testing
  - Import an empty database representing the environment you want to test (we use our Drupal database, but then with all content deleted)
  - Startup a local webserver
  - Update Drupal (make sure the imported database is up to date with the configuration)
  - Run composer's script' behat'
