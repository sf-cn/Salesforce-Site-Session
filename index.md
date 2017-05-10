## Manage session in salesforce site

Manage session in custom force.com site since `Cache.Session` is not available for anonymous users outside community/customer portal/partner portal.

If you need to manage authentication in custom force.com site; use account/contact/custom object as your site user storage. Please review <a href="http://authen.sfdev.cn/"  target="_blank">Custom Authentication Management Package</a>

## Getting started

### Install Managed Package
<a href="https://login.salesforce.com/packaging/installPackage.apexp?p0=04t28000000nuB3" target="_blank">
  Deploy to Production
</a>
<br /><br />
<a href="https://test.salesforce.com/packaging/installPackage.apexp?p0=04t28000000nuB3" target="_blank">
  Deploy to Sandbox
</a>

### Session in Visualforce Page
```APEX
FW.ISession2 session = new FW.PageSession(ApexPages.currentPage());
session.put('name', 'value');
session.CommitSession();
string sessionValue = session.getString('name');
```
* call `CommitSession()` to establish initial session
*  `CommitSession()` can not be call in Virtualforce page controller constructor, the earliest it can be call is in page action method
* always call `CommitSession()` to save session values.
### Session in REST service
```APEX
FW.ISession2 session = new FW.RestSession(RestContext.request, RestContext.response);
session.put('name', 'value');
session.CommitSession();
string sessionValue = session.getString('name');
```
### AJAX to invoke REST service
* HTTP header 'Site.Session' need to be set before send ajax request with the value of `document.cookie`
```JAVASCRIPT
$("#click").on("click", function(){
    axios.get('https://*.csX.force.com/services/apexrest/svc/v1/', {
            method:'get',
            headers: { 'Site.Session': document.cookie }
        })
      .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
});
```
### Sample Controller

```APEX
public class SessionController {

    private FW.ISession2 s;

    // constructor executed first, then page action
    public SessionController() {

        s = new FW.PageSession(ApexPages.currentPage());

        ApexPages.currentPage().getHeaders().put('Cache-Control', 'no-cache, no-store, must-revalidate');
        ApexPages.currentPage().getHeaders().put('Pragma', 'no-cache');
        ApexPages.currentPage().getHeaders().put('Expires', '0');
    }

    // page action is executed after contructor
    public PageReference Authenticate() {

        s.put('name', 'Page Action: - ' + datetime.now().format('yyyy-MM-dd HH:mm:ss', 'Asia/Shanghai'));

        PageReference pageRef;

        if (s.get('anonymous') == null || (boolean) s.get('anonymous') == true) {
            pageRef = new PageReference('/SessionTestLogin');
            pageRef.setRedirect(true);
            s.put('anonymous', true);
        }
        else {
            pageRef = new PageReference('http://google.com');
            pageRef.setRedirect(true);
        }

        s.CommitSession();

        return pageRef;
    }
}
```
### Sample Custom REST API
```APEX
// https://*.csX.force.com/services/apexrest/svc/v1/
@RestResource(urlMapping = '/svc/v1/*')
global without sharing class RestService {

    private static FW.ISession2 s;

    static {

        s = new FW.RestSession(RestContext.request, RestContext.response);
        
        RestContext.response.headers.put('Cache-Control', 'no-cache, no-store, must-revalidate');
        RestContext.response.headers.put('Pragma', 'no-cache');
        RestContext.response.headers.put('Expires', '0');
    }

    @HttpGet
    global static string executeGet() {

        s.put('getDomain()', Site.getDomain());

        s.CommitSession();

        String collegeString = '';
        for (String sss : RestContext.request.headers.keySet()) {
            collegeString += (collegeString == '' ? '' : ',') + sss;
        }

        return collegeString;
    }
}
```
### Sample Virtualforce Page
```HTML
<apex:page controller="SessionController" doctype="html-5.0" applyhtmltag="false" applybodytag="false"
           showheader="false" sidebar="false" standardstylesheets="false">
    <html lang="en">
    <head>
        <!-- Required meta tags -->
        <meta charset="utf-8"></meta>
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no"></meta>

        <!-- Bootstrap CSS -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0-alpha.6/css/bootstrap.min.css" integrity="sha384-rwoIResjU2yc3z8GV/NPeZWAv56rSmLldC3R/AZzGRnGxQQKnKkoFVhFQhNUwEyJ"
              crossorigin="anonymous"></link>

        <title>Session Test</title>

        <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
        <script src="https://code.jquery.com/jquery-3.2.1.min.js" integrity="sha256-hwg4gsxgFZhOsEEamdOYGBf13FyQuiTwlAQgxVSNgt4="
                crossorigin="anonymous"></script>
    </head>
    <body>
        <h1>Session Test</h1>

        <button id="click">Invoke REST Service</button>
        <script>
            $(function(){
                $("#click").on("click", function(){
                    axios.get('https://*.csX.force.com/services/apexrest/svc/v1/', {
                            method:'get',
                            headers: { 'Site.Session': document.cookie }
                        })
                      .then(function (response) {
                        console.log(response);
                      })
                      .catch(function (error) {
                        console.log(error);
                      });
                });
            });
        </script>
    </body>
</html>
</apex:page>
```
