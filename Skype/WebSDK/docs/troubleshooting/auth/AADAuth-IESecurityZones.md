# AAD Auth Failures - Crossing Security Zones in Internet Explorer

_**Applies to:** Skype for Business 2015_

## Who is this article for?

If you are attempting to use the Azure AD authentication option to sign into the Skype for Business Web SDK in Internet Explorer, you are successfully authenticating to Azure AD and getting redirected back to your web application, and then your sign in fails silently or hangs indefinitely, then this might be your issue. If your sign in is also failing in browsers other than IE, this probably is not your issue, or in any case is not your only issue.

This is especially likely to be an issue if you are hosting your application on **http://localhost** during development, because localhost is in the "intranet" security zone by default, while the AAD sign in page is not.

If this is not your issue, you can return to [this page](./AADAuthFailures.md) for a list of other potential issues.

## The Issue

The solution for this issue is very simple, so it might be easier to just see if the solution fixes it rather than spending a long time looking at web traffic to detect the issue.

If you really want to confirm that this is the specific issue you're hitting, you can use Fiddler or Charles to monitor web traffic between your application and the server. If you have this issue, in your web traffic somewhere after authenticating with AAD, redirecting back to your app page, and signing in with `signInManager.signIn`, you will see a request to login.microsoftonline.com that returned a redirect response. Upon  inspecting it, you will see that the fragment of the url itself contains an error, which will say something like this:

> ADSTS50058: A silent sign-in request was sent but no user is signed in. The cookies used to represent the user's session were not sent in the request to Azure AD. This can happen if the user is using Internet Explorer or Edge, and the web app sending the silent sign-in request is in different IE security zone than the Azure AD endpoint

Despite the fact that the error message mentions Microsoft Edge, this issue generally only appears in Internet Explorer.

## The Solution

To fix this issue, you need to somehow ensure that the page where you're hosting your app and the AAD sign in page are treated as though they're in the same security zone.

The recommended way to fix this is to modify your machine's **hosts** file to map some  dummy domain name, eg. **app.myapp.com** to the IP address of localhost (127.0.0.1). To do this on a PC, open **Command Prompt** or another terminal in _administrator mode_. Enter the following command to go to the directory with the hosts file:

`C:\> cd C:/windows/system32/drivers/etc`

Then enter the following to open the hosts file in administrator mode in Notepad:

`C:\> notepad hosts`

Add the following line to the bottom of the file to add an entry mapping **app.myapp.com** to the IP address of localhost:

`127.0.0.1 		app.myapp.com`

Then save the file. If it prompts you to 'save as' rather than just 'save,' you probably did not open the file in administrator mode. You must be in administrator mode to modify the **hosts** file. If you've done this correctly, you should now be able to open a browser, navigate to http://app.myapp.com and see the same page you were hosting before on localhost. However now when you try to sign in, Internet Explorer should treat app.myapp.com as though it is in the "internet" security zone where the AAD sign in page is rather than the "intranet" security zone where localhost is. This should prevent the error where IE  stops cookies from being transported between security zones. You will also have to add the new dummy url to your list of valid reply URLs in the app registration in AAD. For  instructions on how to do that, see [this guide](./AADAuth-ReplyURLs.md). If you have done all this correctly your sign in should complete successfully.

> **Note:** Modifying the **hosts** file is a significant operation and could have an impact on other programs or services. It is recommended that you undo this change after testing and deploying your app.

It also may be possible to fix this by manually listing localhost as as in the "internet" security zone by opening IE, going to **Internet Options > Security > Internet** and adding localhost to the list of sites in that zone. However this is not the recommended method.

## Related Topics:

- [AAD Auth Failures](./AADAuthFailures.md)
- [The reply address 'https://...' does not match the reply addresses configured for the application <...>](./AADAuth-ReplyURLs.md)
