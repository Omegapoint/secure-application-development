# Lab instructions

This describes what you need to run the labs

## Getting started

To set up the environment for the labs you can use either Docker Engine OR node.js (14, 16 or 18)

### Using node.js

```bash
git clone https://github.com/juice-shop/juice-shop.git --depth 1
cd juice-shop
npm install
npm start
```

Browse to [http://localhost:3000](http://localhost:3000)

### Using Docker Engine

```bash
docker pull bkimminich/juice-shop
docker run --rm -p 3000:3000 bkimminich/juice-shop
```

Browse to [http://localhost:3000](http://localhost:3000)

---

## Challenges

Below follows a guide to each challenge. At the end of each challenge we will go through the steps and discuss possible mitigations together.

A general strategy:

* Read the instructions of the challenge carefully
* Think about what we are trying to exploit, and what is possibly wrong!
* If you get stuck, peek at the solution
* If all else fails, you are welcome to ask for help

---

### Broken Access Control

#### Score Board ⭐

Find the hidden score board to track your progress.

<details>
  <summary>Solution</summary>

1. Go to the Sources tab of your browsers DevTools and open the main.js file.
2. If your browser offers pretty-printing of this minified messy code, best use this offer. In Chrome this can be done with the "{}"-button.
3. Search for score and iterate through each finding to come across one looking like a route mapping section:
4. Navigate to http://localhost:3000/#/score-board to solve the challenge.
5. From now on you will see the additional menu item Score Board in the navigation bar.
   
</details>
   
#### Forged Feedback ⭐⭐⭐

The Juice Shop allows users to provide general feedback including a star rating and some free text comment. When logged in, the feedback will be associated with the current user. When not logged in, the feedback will be posted anonymously. This challenge is about vilifying another user by posting a (most likely negative) feedback in his or her name!

* This challenge can be solved via the user interface or by intercepting the communication with the RESTful backend.
* To find the client-side leverage point, closely analyze the HTML form used for feedback submission.
* The backend-side leverage point is similar to some of the XSS challenges found in OWASP Juice Shop.

<details>
  <summary>Solution</summary>

1. Go to the Contact Us form on http://localhost:3000/#/contact.
2. Inspect the DOM of the form in your browser to spot this suspicious text field right at the top:
   `<input _ngcontent-c23 hidden id="userId" type="text" class="ng-untouched ng-pristine ng-valid">`
3. In your browser's developer tools remove the hidden attribute from above \<input\> tag.
4. The field should now be visible in your browser. Type any user's database identifier in there (other than your own if you are currently logged in) and submit the feedback.
  
 </details>

#### Forged Review ⭐⭐⭐

The Juice Shop allows users to provide reviews of all the products. A user has to be logged in before they can post any review for any of the products. This challenge is about vilifying another user by posting a (most likely bad) review in his or her name!

* This challenge can be solved by using developers tool of your browser or with tools like postman.
* Analyze the form used for review submission and try to find a leverage point.
* This challenge is pretty similar to Post some feedback in another user's name challenge.

<details>
  <summary>Solution</summary>

1. Select any product and write a review for it
2. Submit the review while observing the Networks tab of your browser.
3. Analyze the PUT request.
4. Change the author name to admin@juice-sh.op in Request Body and re-send the request.
  
</details>

#### View Basket ⭐⭐

This horizontal privilege escalation challenge demands you to access the shopping basket of another user. Being able to do so would give an attacker the opportunity to spy on the victims shopping behaviour. He could also play a prank on the victim by manipulating the items or their quantity, hoping this will go unnoticed during checkout. This could lead to some arguments between the victim and the vendor.

* Try out all existing functionality involving the shopping basket while having an eye on the HTTP traffic.
* There might be a client-side association of user to basket that you can try to manipulate.
* In case you manage to update the database via SQL Injection so that a user is linked to another shopping basket, the application will not notice this challenge as solved.

<details>
  <summary>Solution</summary>

1. Log in as any user.
2. Put some products into your shopping basket.
3. Inspect the Session Storage in your browser's developer tools to find a numeric bid value.
4. Change the bid, e.g. by adding or subtracting 1 from its value.
5. Visit http://localhost:3000/#/basket to solve the challenge.

If the challenge is not immediately solved, you might have to F5-reload to relay the bid change to the Angular client.
  
</details>

#### Manipulate Basket ⭐⭐⭐

View another user's shopping basket was only about spying out other customers. For this challenge you need to get your hands dirty by putting a product into someone else's basket that cannot be already in there!

* Check the HTTP traffic while placing products into your own shopping basket to find a leverage point.
* Adding more instances of the same product to someone else's basket does not qualify as a solution. The same goes for stealing from someone else's basket.
* This challenge requires a bit more sophisticated tampering than others of the same ilk.

<details>
  <summary>Solution</summary>

1. Log in as any user.
2. Inspect HTTP traffic while putting items into your own shopping basket to learn your own BasketId. For this solution we assume yours is 1 and another user's basket with a BasketId of 2 exists.
3. Submit a POST request to http://localhost:3000/api/BasketItems with payload as

    ```json
    {
        "ProductId": 14,
        "BasketId": "2",
        "quantity": 1
    }
    ```

   making sure no product of that with ProductId of 14 is already in the target basket. Make sure to supply your Authorization Bearer token in the request header.
4. You will receive a (probably unexpected) response of `{'error' : 'Invalid BasketId'}` - after all, it is not your basket!
5. Change your POST request into utilizing HTTP Parameter Pollution (HPP) by supplying your own BasketId and that of someone else in the same payload, i.e.

    ```json
    {
        "ProductId": 14,
        "BasketId": "1",
        "quantity": 1,
        "BasketId": "2"
    }
    ```

6. Submitting this request will satisfy the validation based on your own BasketId but put the product into the other basket!
  
</details>

#### Exposed Metrics ⭐

The popular monitoring system being referred to in the challenge description is Prometheus.

* The Juice Shop serves its metrics on the default path expected by Prometheus
* Guessing the path is probably just as quick as taking the RTFM route via https://prometheus.io/docs/introduction/first_steps

<details>
  <summary>Solution</summary>

1. Scroll through https://prometheus.io/docs/introduction/first_steps
2. You should notice several mentions of /metrics as the default path scraped by Prometheus, e.g. "Prometheus expects metrics to be available on targets on a path of /metrics."
3. Visit http://localhost:3000/metrics to view the actual Prometheus metrics of the Juice Shop and solve this challenge
  
</details>

---

### Cryptographic Failures

#### Easter Egg ⭐⭐⭐⭐

There is a hidden easter egg in the `/ftp` directory. Try to download it!

When you open the easter egg file, you might be a little disappointed, as the developers taunt you about not having found the real easter egg! Of course finding that is a follow-up challenge to this one.

<details>
  <summary>Solution</summary>

1. Use the Poison Null Byte attack
2. ...to download http://localhost:3000/ftp/eastere.gg%2500.md
  
</details>

#### Nested Easter Egg ⭐⭐⭐⭐

Solving the Find the hidden easter egg challenge was probably no as satisfying as you had hoped. Now it is time to tackle the taunt of the developers and hunt down the real easter egg. This follow-up challenge is basically about finding a secret URL that - when accessed - will reward you with an easter egg that deserves the name.

* Make sure you solve Find the hidden easter egg first.
* You might have to peel through several layers of tough-as-nails encryption for this challenge.

<details>
  <summary>Solution</summary>

1. Get the encrypted string from the eastere.gg from the Find the hidden easter egg challenge: `L2d1ci9xcmlmL25lci9mYi9zaGFhbC9ndXJsL3V2cS9uYS9ybmZncmUvcnR0L2p2Z3V2YS9ndXIvcm5mZ3JlL3J0dA==`
2. Base64-decode this into `/gur/qrif/ner/fb/shaal/gurl/uvq/na/rnfgre/rtt/jvguva/gur/rnfgre/rtt`
3. Trying this as a URL will not work. Notice the recurring patterns (rtt, gur etc.) in the above string
4. ROT13-decode this into `/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg`
5. Visit http://localhost:3000/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg
  
</details>

#### Access Log ⭐⭐⭐⭐

The Juice Shop application server is writing access logs, which can contain interesting information that competitors might also be interested in.

* Normally, server log files are written to disk on server side and are not accessible from the outside.
* Which raises the question: Who would want a server access log to be accessible through a web application?
* One particular file found in the folder you might already have found during the Access a confidential document challenge might give you an idea who is interested in such a public exposure.
* Drilling down one level into the file system might not be sufficient.

<details>
  <summary>Solution</summary>

1. Solve the Access a confidential document or any related challenges which will bring the exposed /ftp folder to your attention.
2. Visit http://localhost:3000/ftp and notice the file incident-support.kdbx which is needed for Log in with the support team's original user credentials and indicates that some support team is performing its duties from the public Internet and possibly with VPN access.
3. Guess luckily or run a brute force attack with e.g. OWASP ZAPs DirBuster plugin for a possibly exposed directory containing the log files.
4. Following the hint to drill down deeper than one level, you will at some point end up with http://localhost:3000/support/logs.
5. Inside you will find at least one access.log of the current day. Open or download it to solve this challenge.
  
</details>

#### GDPR Data Theft ⭐⭐⭐⭐

In order to comply with GDPR, the Juice Shop offers a Request Data Export function for its registered customers. It is possible to exploit a flaw in the feature to retrieve more data than intended. Injection attacks will not count to solve this one.

* You should not try to steal data from a "vanilla" user who never even ordered something at the shop.
* As everything about this data export functionality happens on the server-side, it won't be possible to just tamper with some HTTP requests to solve this challenge.
* Inspecting various server responses which contain user-specific data might give you a clue about the mistake the developers made.

<details>
  <summary>Solution</summary>

1. Log in as any user, put some items into your basket and create an order from these.
2. Notice that you end up on a URL with a seemingly generated random part, like http://localhost:3000/#/order-completion/5267-829f123593e9d098
3. On that Order Summary page, click on the Track Orders link under the Thank you for your purchase! message to end up on a URL simular to http://localhost:3000/#/track-result/new?id=5267-829f123593e9d098
4. Open the network tab of your browser's DevTools and refresh that page. You should notice a request similar to http://localhost:3000/rest/track-order/5267-829f123593e9d098.
5. Inspecting the response closely, you might notice that the user email address is partially obfuscated: {"status":"success","data":[{"orderId":"5267-829f123593e9d098","email":"*dm*n@j**c*-sh.*p","totalPrice":2.88,"products":[{"quantity":1,"name":"Apple Juice (1000ml)","price":1.99,"total":1.99,"bonus":0},{"quantity":1,"name":"Apple Pomace","price":0.89,"total":0.89,"bonus":0}],"bonus":0,"eta":"2","_id":"tosmfPsDaWcEnzRr3"}]}
6. It looks like certain letters - seemingly all vowels - were replaced with * characters before the order was stored in the database.
7. Register a new user with an email address that would result in the exact same obfuscated email address. For example register edmin@juice-sh.op to steal the data of admin@juice-sh.op.
8. Log in with your new user and immediately get your data exported via http://localhost:3000/#/privacy-security/data-export.
9. You will notice that the order belonging to the existing user admin@juice-sh.op (in this example 5267-829f123593e9d098) is part of your new user's data export due to the clash when obfuscating emails!

</details>
  
---

### Injection

#### Login Admin ⭐⭐

What would a vulnerable web application be without an administrator user account whose (supposedly) privileged access rights a successful hacker can abuse?

* The challenge description probably gave away what form you should attack: the login form
* If you happen to know the email address of the admin already, you can launch a targeted attack.
* You might be lucky with a dedicated attack pattern even if you have no clue about the admin email address.

Note: It is possible to solve this without applying SQL Injection. You can, for exampel, simply try guessing the password. But do try to find the injection!

<details>
  <summary>Solution</summary>
  
* With the email, append `'--` in the email-field and use any password
* Wihtout the email, in the email-field, use `' or 1=1--` and any password
  * This will automatically authenticate the first entry in the `Users` table, which coincidentally happens to be the administrator
  
</details>

#### Database Schema ⭐⭐⭐

An attacker would try to exploit SQL Injection to find out as much as possible about your database schema. This subsequently allows much more targeted, stealthy and devastating SQL Injections, like Retrieve a list of all user credentials via SQL Injection.

* Find out which database system is in use and where it would usually store its schema definitions.
* Craft a UNION SELECT attack string to join the relevant data from any such identified system table into the original result.
* You might have to tackle some query syntax issues step-by-step, basically hopping from one error to the next
* As with Order the Christmas special offer of 2014 this cannot be achieved through the application frontend.

<details>
  <summary>Solution</summary>

1. From any errors seen during previous SQL Injection attempts you should know that SQLite is the relational database in use.
2. Check https://www.sqlite.org/faq.html to learn in "(7) How do I list all tables/indices contained in an SQLite database" that the schema is stored in a system table `sqlite_master`.
3. You will also learn that this table contains a column `sql` which holds the text of the original `CREATE TABLE` or `CREATE INDEX` statement that created the table or index. Getting your hands on this would allow you to replicate the entire DB schema.
4. During the Order the Christmas special offer of 2014 challenge you learned that the `/rest/products/search` endpoint is susceptible to SQL Injection into the `q` parameter.
5. The attack payload you need to craft is a `UNION SELECT` merging the data from the `sqlite_master` table into the products returned in the JSON result.
6. As a starting point we use the known working `'))--` attack pattern and try to make a `UNION SELECT` out of it
7. Searching for `')) UNION SELECT * FROM x--` fails with a `SQLITE_ERROR: no such table: x` as you would expect.
8. Searching for `')) UNION SELECT * FROM sqlite_master--` fails with a promising `SQLITE_ERROR: SELECTs to the left and right of UNION do not have the same number of result columns` which least confirms the table name.
9. The next step in a `UNION SELECT`-attack is typically to find the right number of returned columns. As the Search Results table in the UI has 3 columns displaying data, it will probably at least be three. You keep adding columns until no more `SQLITE_ERROR` occurs (or at least it becomes a different one):

    * `')) UNION SELECT '1' FROM sqlite_master--` fails with `number of result columns` error
    * `')) UNION SELECT '1', '2' FROM sqlite_master--` fails with `number of result columns` error
    * `')) UNION SELECT '1', '2', '3' FROM sqlite_master--` fails with `number of result columns` error
    * (...)
    * `')) UNION SELECT '1', '2', '3', '4', '5', '6', '7', '8' FROM sqlite_master--` still fails with `number of result columns` error
    * `')) UNION SELECT '1', '2', '3', '4', '5', '6', '7', '8', '9' FROM sqlite_master--` finally gives you a JSON response back with an extra element `{"id":"1","name":"2","description":"3","price":"4","deluxePrice":"5","image":"6","createdAt":"7","updatedAt":"8","deletedAt":"9"}`.

10. Next you get rid of the unwanted product results changing the query into something like `qwert')) UNION SELECT '1', '2', '3', '4', '5', '6', '7', '8', '9' FROM sqlite_master--` leaving only the "`UNION`ed" element in the result set
11. The last step is to replace one of the fixed values with correct column name `sql`, which is why searching for `qwert')) UNION SELECT sql, '2', '3', '4', '5', '6', '7', '8', '9' FROM sqlite_master--` solves the challenge.
  
</details>

#### User Credentials ⭐⭐⭐⭐

This challenge explains how a considerable number of companies were affected by data breaches without anyone breaking into the server room or sneaking out with a USB stick full of sensitive information. Given your application is vulnerable to a certain type of SQL Injection attacks, hackers can have the same effect while comfortably sitting in a café with free WiFi.

* Try to find an endpoint where you can influence data being retrieved from the server.
* Craft a UNION SELECT attack string to join data from another table into the original result.
* You might have to tackle some query syntax issues step-by-step, basically hopping from one error to the next
* As with Order the Christmas special offer of 2014 and Exfiltrate the entire DB schema definition via SQL Injection this cannot be achieved through the application frontend.

<details>
  <summary>Solution</summary>

1. During the Database Schema challenge you learned that the `/rest/products/search` endpoint is susceptible to SQL Injection into the `q` parameter.
2. Searching for `qwert')) UNION SELECT id, email, password, '4', '5', '6', '7', '8', '9' FROM Users--` solves the challenge giving you a the list of all user data in convenient JSON format.
  
</details>

#### DOM XSS ⭐

Look for an XSS in the search box!

<details>
  <summary>Solution</summary>

1. Paste the attack string `<iframe src="javascript:alert(`xss`)">` into the Search... field.
2. Hit the Enter key.
3. An alert box with the text "xss" should appear.
  
</details>

#### Reflected XSS ⭐⭐

Look for a URL parameter where its value appears on the page it is leading to
Try probing for XSS vulnerabilities by submitting text wrapped in an HTML tag which is easy to spot on screen, e.g. `<h1>` or `<strike>`.

<details>
  <summary>Solution</summary>

1. Log in as any user.
2. Do some shopping and then visit the Order History.
3. Clicking on the little "Truck" button for any of your orders will show you the delivery status of your order.
4. Notice the id parameter in the URL http://localhost:3000/#/track-result?id=fe01-f885a0915b79f2a9 with fe01-f885a0915b79f2a9 being one of your order numbers?
5. As the fe01-f885a0915b79f2a9 is displayed on the screen, it might be susceptible to an XSS attack.
6. Paste the attack string `<iframe src="javascript:alert(`xss`)">` into that URL so that you have http://localhost:3000/#/track-result?id=%3Ciframe%20src%3D%22javascript:alert(%60xss%60)%22%3E
7. Refresh that URL to get the XSS payload executed and the challenge marked as solved.
  
</details>

#### Server-side XSS Protection

This is one of the hardest XSS challenges, as it cannot be solved by just fiddling with the client-side JavaScript or bypassing the client entirely. Whenever there is a server-side validation or input processing involved, you should investigate how it works. Finding out implementation details e.g. used libraries, modules or algorithms - should be your priority. If the application does not leak this kind of details, you can still go for a blind approach by testing lots and lots of different attack payloads and check the reaction of the application.

When you actually understand a security mechanism you have a lot higher chance to beat or trick it somehow, than by using a trial and error approach.

* The Comment field in the Contact Us screen is where you want to put your focus on
* The attack payload `<iframe src="javascript:alert(`xss`)">` will not be rejected by any validator but stripped from the comment before persisting it
* Look for possible dependencies related to input processing in the package.json.bak you harvested earlier
* If an xss alert shows up but the challenge does not appear as solved on the Score Board, you might not have managed to put the exact attack string `<iframe src="javascript:alert(`xss`)">` into the database?

<details>
  <summary>Solution</summary>

In the package.json.bak you might have noticed the pinned dependency "sanitize-html": "1.4.2". Internet research will yield a reported Cross-site Scripting (XSS) vulnerability, which was fixed with version 1.4.3 - one release later than used by the Juice Shop. The referenced GitHub issue explains the problem and gives an exploit example:

> Sanitization is not applied recursively, leading to a vulnerability to certain masking attacks. Example:
> I am not harmless: `<<img src="csrf-attack"/>img src="csrf-attack"/>` is sanitized to I am not harmless: `<img src="csrf-attack"/>`
> Mitigation: Run sanitization recursively until the input html matches the output html.

1. Visit http://localhost:3000/#/contact.
2. Enter `<<script>Foo</script>iframe src="javascript:alert(`xss`)">` as Comment
3. Choose a rating and click Submit
4. Visit http://localhost:3000/#/about for a first "xss" alert (from the Customer Feedback slideshow)
5. Visit http://localhost:3000/#/administration for a second "xss" alert (from the Customer Feedback table)

</details>
