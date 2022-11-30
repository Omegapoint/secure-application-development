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

### Insecure Design

#### Threat Modeling Workshop ⭐

TODO

---

### Security Misconfiguration

#### Error Handling ⭐

Provoke an error that is neither very gracefully nor consistently handled.

<details>
  <summary>Solution</summary>

  Any request that cannot be properly handled by the server will eventually be passed to a global error handling component that sends an error page to the client that includes a stack trace and other sensitive information. The restful API behaves similarly, passing back a JSON error object with sensitive data, such as SQL query strings.

  Here are two examples (out of many ways) to provoke such an error situation and solve this challenge immediately:

  * Log in to the application with `'` (single-quote) as Email and anything as Password
  * Visit http://localhost:3000/rest/qwerty

</details>

#### Deprecated Interface ⭐⭐

Use a deprecated B2B interface that was not properly shut down.

<details>
  <summary>Solution</summary>

  1. Log in as any user.
  2. Click Complain? in the Contact Us dropdown to go to the File Complaint form
  3. Clicking the file upload button for Invoice and browsing some directories you might notice that `.pdf` and `.zip` files are filtered by default
  4. Trying to upload another other file will probably give you an error message on the UI stating exactly that: `Forbidden file type. Only PDF, ZIP allowed.`
  5. Open the `main.js` in your DevTools and find the declaration of the file upload (e.g. by searching for `zip`)
  6. In the `allowedMimeType` array you will notice `"application/xml"` and `"text/xml"` along with the expected PDF and ZIP types
  7. Click on the Choose File button.
  8. In the File Name field enter `*.xml` and select any arbitrary XML file (<100KB) you have available. Then press Open.
  9. Enter some Message text and press Submit to solve the challenge.
  10. On the JavaScript Console of your browser you will see a suspicious `410 (Gone)` HTTP Error. In the corresponding entry in the Network section of your browser's DevTools, you should see an error message, telling you that `B2B customer complaints via file upload have been deprecated for security reasons!`

</details>

---

### Vulnerable and Outdated Components

#### Vulnerable Library ⭐⭐⭐⭐

Inform the shop about a vulnerable library it is using. (Mention the exact library name and version in your comment)

<details>
<summary>Solution</summary>

  Juice Shop depends on a JavaScript library with known vulnerabilities. Having the package.json.bak and using an online vulnerability database like Retire.js or a CLI tool like npm-audit that comes with Node.js, makes it rather easy to identify it.

  1. Access a developer's forgotten backup file [http://localhost:3000/ftp/package.json.bak%2500.md](http://localhost:3000/ftp/package.json.bak%2500.md)
  2. Checking the dependencies in package.json.bak for known vulnerabilities online will give you a match.
  3. Visit [http://localhost:3000/#/contact](http://localhost:3000/#/contact)
  4. Submit feedback with the sting pair `sanitize-html` and `1.4.2` or `express-jwt` and `0.1.3`.

</details>

#### Unsigned JWT ⭐⭐⭐⭐⭐

Forge an essentially unsigned JWT token that impersonates the (non-existing) user jwtn3d@juice-sh.op.

<details>
<summary>Solution</summary>

  1. Log in as any user to receive a valid JWT in the `Authorization` header.
  2. Copy the JWT (i.e. everything after `Bearer` in the `Authorization` header) and decode it.
  3. Under the `payload` property, change the `email` attribute in the JSON to `jwtn3d@juice-sh.op`.
  4. Change the value of the `alg` property in the `header` part from `HS256` to `none`.
  5. Encode the `header` to `base64url`. Similarly, encode the `payload` to `base64url`. `base64url` makes it URL safe, a regular Base64 encode might not work!
  6. Join the two strings obtained above with a `.` (dot symbol) and add a `.` at the end of the obtained string. So, effectively it becomes `base64url(header).base64url(payload).`
  7. Change the `Authorization` header of a subsequent request to the retrieved JWT (prefixed with `Bearer` as before) and submit the request. Alternatively you can set the `token` cookie to the JWT which be used to populate any future request with that header.

</details>

---

### Identification and Authentication Failures

#### Log in with the administrator's user account ⭐⭐

Log in with the administrator's user credentials without previously changing them or applying SQL Injection.

<details>
<summary>Solution</summary>

1. Crack the password of the adminstrator user `0192023a7bbd73250516f069df18b500`.
2. You can use [CrackStation](https://crackstation.net/), `john` or `hashcat`.
3. Log in with Email `admin@juice-sh.op` and Password `admin123`.

</details>

#### Reset Jim's Password ⭐⭐

Reset Jim's password via the Forgot Password mechanism with the truthful answer to his security question.

<details>
<summary>Solution</summary>

  1. Visit http://localhost:3000/#/forgot-password and provide jim@juice-sh.op as your Email to learn that Your eldest siblings middle name? is Jim's chosen security question
  2. Jim (whose UserId happens to be 2) left some breadcrumbs in the application which reveal his identity

      * A product review for the OWASP Juice Shop-CTF Velcro Patch stating "Looks so much better on my uniform than the boring Starfleet symbol."
      * Another product review "Fresh out of a replicator" on the Green Smoothie product
      * A Recycling Request associated to his saved address "Room 3F 121, Deck 5, USS Enterprise, 1701"

  3. It should eventually become obvious that James T. Kirk is the only viable solution to the question of Jim's identity
  4. Visit https://en.wikipedia.org/wiki/James_T._Kirk and read the Depiction section
  5. It tells you that Jim has a brother named George Samuel Kirk
  6. Visit http://localhost:3000/#/forgot-password and provide jim@juice-sh.op as your Email
  7. In the subsequently appearing form, provide Samuel as Your eldest siblings middle name?
  8. Then type any New Password and matching Repeat New Password
  9. Click Change to solve this challenge

</details>

#### Reset Bender's Password ⭐⭐

Reset Bender's password via the Forgot Password mechanism with the truthful answer to his security question.

<details>
<summary>Solution</summary>

  1. Trying to find out who "Bender" might be should immediately lead you to Bender from Futurama as the only viable option
  2. Visit https://en.wikipedia.org/wiki/Bender_(Futurama) and read the Character Biography section
  3. It tells you that Bender had a job at the metalworking factory, bending steel girders for the construction of suicide booths.
  4. Find out more on Suicide Booths on http://futurama.wikia.com/wiki/Suicide_booth
  5. This site tells you that their most important brand is `Stop'n'Drop`
  6. Visit http://localhost:3000/#/forgot-password and provide `bender@juice-sh.op` as your Email
  7. In the subsequently appearing form, provide `Stop'n'Drop` as Company you first work for as an adult?
  8. Then type any New Password and matching Repeat New Password
  9. Click Change to solve this challenge

</details>
