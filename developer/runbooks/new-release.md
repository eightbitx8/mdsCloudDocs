# Creating a new release of a MDS Cloud component

* `git status` to ensure no modifications locally before building and publishing
* `docker login` for mdsCloud account
* `docker build -t mdscloud/mds-state-machine:latest . && docker build -t mdscloud/mds-state-machine:0.0.1 . && docker push mdscloud/mds-state-machine:latest && docker push mdscloud/mds-state-machine:0.0.1`
* `git tag -a v0.0.1`
    * Push of version 0.0.1 to docker hub