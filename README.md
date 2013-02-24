# createsend-java [![Build Status](https://secure.travis-ci.org/campaignmonitor/createsend-java.png)][travis]

[travis]: http://travis-ci.org/campaignmonitor/createsend-java

A Java library which implements the complete functionality of the [Campaign Monitor API](http://www.campaignmonitor.com/api/).

## Authenticating

The Campaign Monitor API supports authentication using either OAuth or an API key.

### Using OAuth

This library helps you authenticate using OAuth, as described in the Campaign Monitor API [documentation](http://www.campaignmonitor.com/api/getting-started/#authenticating_with_oauth). You may also wish to reference this Java [example application](https://gist.github.com/jdennes/4997342). The authentication process is described below.

The first thing your application should do is redirect your user to the Campaign Monitor authorization URL where they will have the opportunity to approve your application to access their Campaign Monitor account. You can get this authorization URL by using `com.createsend.General.getAuthorizeUrl()`, like so:

```java
String authorizeUrl = com.createsend.General.getAuthorizeUrl(
    32132,                 // The Client ID for your application
    "982u39823r928398",    // The Client Secret for your application
    "http://example.com/", // Redirect URI for your application
    "ViewReports",         // The permission level your application requires
    "some state data"      // Optional state data to be included
);
// Redirect your users to authorizeUrl.
```

If your user approves your application, they will then be redirected to the redirect URI you specified, which will include a `code` parameter, and optionally a `state` parameter in the query string. Your application should implement a handler which can exchange the code passed to it for an access token, using `com.createsend.General.exchangeToken()` like so:

```java
com.createsend.models.OAuthTokenDetails tokenDetails = com.createsend.General.exchangeToken(
    32132,                 // The Client ID for your application
    "982u39823r928398",    // The Client Secret for your application
    "http://example.com/", // Redirect URI for your application
    "8dw9uq98wu98d"        // The unique code for your user found in the query string
);
// Save your access token, 'expires in' value, and refresh token (in tokenDetails).
```

At this point you have an access token and refresh token for your user which you should store somewhere convenient so that your application can look up these values when your user wants to make future Campaign Monitor API calls.

Once you have an access token and refresh token for your user, you can authenticate and make further API calls like so:

```java
import com.createsend.General;
import com.createsend.models.clients.ClientBasics;
import com.createsend.util.OAuthAuthenticationDetails;
import com.createsend.util.exceptions.CreateSendException;

public class Tester {
    public static void main(String[] args) throws CreateSendException {
        OAuthAuthenticationDetails auth = new OAuthAuthenticationDetails(
            "your access token", "your refresh token");
        General general = new General(auth);
        ClientBasics[] clients = general.getClients();
    }
}
```

All OAuth tokens have an expiry time, and can be renewed with a corresponding refresh token. If your access token expires when attempting to make an API call, a `com.createsend.util.exceptions.ExpiredOAuthTokenException` will be thrown, so your code should handle this. Here's an example of how you could do this:

```java
import com.createsend.General;
import com.createsend.models.clients.ClientBasics;
import com.createsend.util.OAuthAuthenticationDetails;
import com.createsend.util.exceptions.CreateSendException;
import com.createsend.util.exceptions.ExpiredOAuthTokenException;

public class Tester {
    public static void main(String[] args) throws CreateSendException {
        OAuthAuthenticationDetails auth = new OAuthAuthenticationDetails(
            "your access token", "your refresh token");

        General general = new General(auth);
        ClientBasics[] clients;

        try {
            clients = general.getClients();
        } catch (ExpiredOAuthTokenException ex) {
            OAuthTokenDetails newTokenDetails = general.refreshToken();
            // Save your updated access token, 'expires in' value, and refresh token
            clients = general.getClients(); // Make the call again
        }
    }
}
```

### Using an API key

```java
import com.createsend.General;
import com.createsend.models.clients.ClientBasics;
import com.createsend.util.ApiKeyAuthenticationDetails;
import com.createsend.util.exceptions.CreateSendException;

public class Tester {
    public static void main(String[] args) throws CreateSendException {
        ApiKeyAuthenticationDetails auth = new ApiKeyAuthenticationDetails(
            "your api key");
        General general = new General(auth);
        ClientBasics[] clients = general.getClients();
    }
}
```

## Basic usage

This example of listing all your clients and their campaigns demonstrates basic usage of the library and the data returned from the API:

```java
import com.createsend.Clients;
import com.createsend.General;
import com.createsend.models.campaigns.SentCampaign;
import com.createsend.models.clients.ClientBasics;
import com.createsend.util.OAuthAuthenticationDetails;
import com.createsend.util.exceptions.CreateSendException;

public class Tester {
    public static void main(String[] args) throws CreateSendException {
        OAuthAuthenticationDetails auth = new OAuthAuthenticationDetails(
            "your access token", "your refresh token");
        General general = new General(auth);
        General general = new General(auth);
        ClientBasics[] clients = general.getClients();

        for (ClientBasics cl : clients) {
            System.out.printf("Client: %s\n", cl.Name);
            Clients cls = new Clients(auth, cl.ClientID);
            System.out.printf("- Campaigns:\n");
            for (SentCampaign cm : cls.sentCampaigns()) {
                System.out.printf("  - %s\n", cm.Subject);
            }
        }
    }
}
```

See the [samples](https://github.com/campaignmonitor/createsend-java/blob/master/samples/com/createsend/samples/SampleRunner.java) directory for more example code.

## Documentation
Full javadoc for this library is available [here](http://campaignmonitor.github.com/createsend-java/doc/).

## Building

### Use Gradle:
Run the following command from the root directory of the repository:

```
gradle -i
```

### Developing with Eclipse:
Gradle can be used to create the `.classpath` and `.project` files to import the project into Eclipse. Run the following command from the root directory of the repository:

```
gradle eclipse
```

### Developing with IDEA
Gradle can be used to create an IDEA project and module files. Run the following command from the root directory of the repository:

```
gradle idea
```

## Contributing
1. Fork the repository
2. Make your changes.
3. Ensure that the build passes, by running `gradle -i` (CI runs on: `openjdk6`, `openjdk7`, and `oraclejdk7`)
4. It should go without saying, but do not increment the version number in your commits.
5. Submit a pull request.
