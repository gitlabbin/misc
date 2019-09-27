Keycloak on OpenShift
---------------------
Thursday, May 31 2018, posted by Stian Thorgersen
https://www.keycloak.org/2018/05/keycloak-on-openshift.html

In this post you'll see how to deploy Keycloak on OpenShift. You'll also learn how to deploy a Node.js based REST service and an HTML5 application to OpenShift and secure these with Keycloak.

There is also a screencast showing this example at https://youtu.be/9zUWqbK3BqI.

If you don't already have OpenShift available a good place to start is by using MiniShift.

## Deploying Keycloak
First of all create a new project in OpenShift with oc by running:

    oc new-project keycloak

The next thing to do is to import the Keycloak template into OpenShift, by running:

```bash
oc replace --force -f "https://raw.githubusercontent.com/jboss-dockerfiles/keycloak"\
"/master/openshift-examples/keycloak-https.json"
```

Now open the OpenShift console and open the keycloak project.

Click on Add to Project and Browse Catalog. In the catalog you should find Keycloak. Click on it.

Click next on the information. Under configuration set a username and password that you can remember in the Keycloak Administrator Username and Keycloak Administrator Password fields. Then click on create. Click on Continue to project overview.

Wait for the deployment to complete then click on the link to the application. Your browser will complain about the certificate as it is a self-signed certificate. Ignore this and proceed. Click on Administration Console, then login with the username and password you entered previously. Keep this tab open as you will need it later.

You have now deployed Keycloak onto OpenShift.

## Configure Clients in Keycloak
We need to create clients for the service and the application we will secure.

Open the tab with the Keycloak admin console. Click on Clients and Create. For Client ID enter service and click Save. Under Access Type select bearer-only and click on Save.

Click on Clients then Create again. For Client ID enter app and click Save. For Valid Redirect URIs and Web Origins enter *. In production environment it is very important that you enter the correct URL for your application, but since this is a demonstration we will simply allow all URLs for simplicity. You can easily update these to the correct URLs for the application after it has been deployed.

Keep the Keycloak admin console tab open as again you will need it later.

### Deploy the Service
Go back to the tab with the OpenShift console and click on Add to Project and Browse Catalog again. This time click on Node.js. Click next on Information, then click on advanced options under Configuration.

Make the following changes:

- Name: service
- Git Repository URL: https://github.com/stianst/misc.git
- Context Dir: openshift/service
- Secure route: enable
- TLS Termination: Edge
- Insecure Traffic: Redirect
- Deployment Config
  * KEYCLOAK_URL=https://secure-keycloak-keycloak.192.168.42.52.nip.io/auth

Replace the value for KEYCLOAK_URL with the URL for Keycloak. You can find this by going back to the tab with the Keycloak admin console (copy the URL up to and including "/auth").
Click on Create then Continue to the project overview. Wait for the build and deployment to complete then click on the link to the application. You should see "Not found!". Add "/service/public" to the url and you should see "message: public" in JSON.

You have now deployed and secured the service. Keep this tab open as well as you need it later.

### Deploy the Application
Go back to the tab with the OpenShift console and click on Add to Project and Browse Catalog again. This time click on PHP. Click next on Information, then click on advanced options under Configuration.

Make the following changes:

- Name: app
- Git Repository URL: https://github.com/stianst/misc.git
- Context Dir: openshift/app
- Secure route: enable
- TLS Termination: Edge
- Insecure Traffic: Redirect
- Deployment Config
  * KEYCLOAK_URL=https://secure-keycloak-keycloak.192.168.42.52.nip.io/auth
  * SERVICE_URL=https://service-keycloak.192.168.42.240.nip.io/service

Replace the value for KEYCLOAK_URL with the URL for Keycloak. You can find this by going back to the tab with the Keycloak admin console (copy the URL up to and including "/auth"). Also, replace the value for SERVICE_URL with the URL for the Service. You can find this by going back to the tab with the service (copy the URL up to and including "/service").
Click on Create then Continue to the project overview. Wait for the build and deployment to complete then click on the link to the application. You should already be logged-in. You can now invoke the service by clicking on Invoke Public to invoke the unsecured endpoint or Invoke Admin to invoke the endpoint secured with the admin role. If you click on Invoke Secured it will fail as the admin user you are logged in with does not have the user role. To be able to invoke this endpoint as well go back to the Keycloak admin console. Create a realm role named user. Then go to users find your admin user and under role mappings add the user role to the user.

You have now deployed and secured the application as well as seen how the application can securely invoke the service you deployed previously.
