# Sandbox Migration to 1.1.5.5 Guide

## Introduction
Below are the steps which are needed to be followed for migrating to the required version 1.1.5

* Infra Changes
	- Update the open source Infra repo 1.1.5.4 branch into the required existing version branch of forked repo.
	- update/crosscheck the secrets.yml in the current branch and resolve all the conflicts.
	- UPDATE ```config_repo``` section all.yml with details pointing to the latest existing branch.
	- Use syncdata service 1.1.5.3 version when after migration so that existing reg-client with 1.1.4.x version and 1.1.5.x version both can be synced a the same time.
        - Update the postgres upgrade section in all.yml as required.
          ```
            upgrade:
            set: true  # accepts true or false value as per need.
            version: 1.1.5 # version gto which upgrade is needed.
            type: release
          ```

* Config Changes
	- Rebase the 1.1.5.4 open source config into the current required existing branch.
	- update the ciphered text for password changes if any.

* DB Release Changes
	- Steps to execute is given in respective DB section in Modular repositories. ``` eg. for mosip_master, mosip_websub check Commons ```
         OR
        - Execute the ``` postgres-init.yml ``` playbook after updating the upgrade section in ``` all.yml ``` (as mentioned in Infra change section).

* Nginx changes
        - 1.1.5.* version has some changes for the below points so we need to redeploy nginx once after updating sandbox ```domain name``` in ```all.yml```.
        - minio nodeport exposing over console VM (in case using sandbox minio).
        - Pms module new api's
        - Command to redeploy using playbooks from ```sb``` folder is
        - ```an playbooks/nginx.yml```

* Minio Migration for moving all the packets to one bucket taken as part of build 1.1.4.3 and above.
	- Remove the minio service running in the MZ cluster using the command ```helm1 delete minio```.
	- Redeploy the minio service in the MZ cluster with ansible command ```an playbooks/minio.yml``` from ```sb``` folder.
	- Open port 32000 on the console so that the minio service can be accessed over nodeport for tcp connection on this port.
	- Please check if above mentioned nginx changes are already done before performing this minio migration.
	- Follow the instruction as given in ```Readme.md``` in the minio migraton section in utils of infra ```utils/minio_migration```.

* Deploying Latest Mosip Modules
	- Uninstall all the mosip modules from mz and dmz clusters 
	- Update artifactory to the latest version i.e 1.1.5 in versions.yml and redeploy.
	- Install all the mz and dmz MOSIP modular services.

## Note
In MOISP the encryption/decryption keys are mapped against the set of application ID and reference ID. There is a change in application ID and reference ID being used for encryption/decryption in pre-registration and id-repository from 1.1.4 to 1.1.5.x.

In pre-registration, for encryption of demographic data, application ID, "REGISTRATION" and reference ID "" (blank) was used and now for encryption application ID "PRE_REGISTRATION" and reference ID "INDIVIDUAL" is used. Hence, to proceed ahead, we need to build a utility to migrate the data created in 1.1.4.

In id-repository, for encryption of demographic data, uin, biometric data and document we were earlier using the application ID, "IDREPOSITORY" and reference ID "" (blank). But now we are using the same application ID but different reference IDs which is configured based on the below properties in id-repository-mz.properties.

* mosip.idrepo.crypto.refId.uin=uin (for UIN number)
* mosip.idrepo.crypto.refId.uin-data=identity_data (for demographic data)
* mosip.idrepo.crypto.refId.demo-doc-data=demographic_data (for documents)
* mosip.idrepo.crypto.refId.bio-doc-data=biometric_data (for biometrics)

Hence, to proceed ahead, we can build a utility to migrate the data or mark the configurations as "" (blank).
