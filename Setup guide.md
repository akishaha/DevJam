#Setup instructions

##Pre-requisites

* All attendees must have an Apigee Edge Developer Account - [sign up here] (https://accounts2.apigee.com/accounts/sign_up?int_medium=website&int_campaign=signup&int_source=follow)
*	Create an org in BaaS. We will use BaaS as a datastore throughout the labs.
*	Provision DevJam attendees into both Edge & BaaS Orgs with Org Administrator role.
* Attendees will need the URL to access the github repository with the lab material. As the instructions are in markdown it's best for them just to view it direct on the github website.
* NOTE: to download a file from github, open the file, and right click, save as... on the RAW button in the top right of the file window

##Configure BaaS

*	[Login](https://apibaas.apigee.com) to BaaS management UI
*	Choose your Org and application. We will use Sandbox application for the DevJam.
*	Create a new collection called “hotels”. We will use this collection our labs.
*	Load entities from the “resources/hotels.json” file. You can also create new hotel entities under the hotels collection.
* Set up the guest role with full access to the /hotels (and /hotels/\*) collection.

#Configure Edge
*	[Login](https://apigee.com/edge) to Edge management UI
*	Choose your Org and environment (it is not recommended to use a free org as the performance of the Analytics pieces will cause problems).
*	Import the OAuth (client-credentials) proxy bundle. If you create a new account, you will get this proxy as one of the default bundles. Deploy the proxy into test environment.

##Configure Dev Portal

*	Provision a new Dev Portal
* Configure the Dev Portal to point at your organisation
* Make sure the dev portal has the requirement to provide a call back URL set to optional
* During lab 4, once the attendees have registered themselves as a developer, give them admin permissions to let them complete the lab
