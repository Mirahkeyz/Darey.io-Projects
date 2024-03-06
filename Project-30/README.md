# Working With Jenkins And Jfrog Artifactory

# Create A Local Repository For Docker

- Click on administration, Repository and then Add Repository
![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e290cedc-d16d-45b3-b04d-ab7e98a158fe)

- Since we are exploring Local Repositories, Select Local Repository

- Type Docker in the search box and select the Docker Icon

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c08d4a6a-163f-4098-a5e0-86f44c3f9844)

In the Repository Key box, type in the name of the repository you wish to create for exaple Tooling. so that all the docker images for tooling app can be pushed there

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4ce148be-7c82-4281-ba80-c4e7c330a667)

- Click on the create Local Repository

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5c8f802b-f4f2-4eeb-bdb9-605ecc381264)

- Now you can see that a Docker Repository for tooling has been created

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a14f212e-0b11-45cb-a655-a4049580ed38)

- Create a second Local Repository for Jenkins

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/63dba6a8-4741-431b-a698-c701f2343379)

# Lets Create A Virtual Repository

Remember, a virtual repository aggregates several repositories under a common URL. you can get artifacts from it but you cannot deploy anything to it

- Select virtual repository

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e6fe53d6-eac3-486f-9f2d-7f235bf53b52)

- Name the Virtual Repository as you deem fit, click on the create virtual repository button

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3eac2ca1-fa1a-437c-a5f1-3ed88e63591b)

- Now that the virtual repository is created, it is time add local repositories to it

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/57723f7c-8d32-47b1-8a68-396c7a2f0cb0)

- Scroll down the page to see the local repositories. This is where you select which local repository will be part of the virtual repository. Click on the double arrows to move them

![Snipe10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/46b977f9-8676-4bc5-bdae-def9bae920cc)

- Once Moved you will see them in the included items section

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/641d5622-0853-418c-8892-5bb88bc443f1)

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/bfad5596-b23f-464c-a8e2-eb562ed222a7)
