---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

## Freelance Software Development in Groningen

Hi! I'm Berni Hackl, and I run **Tonnenpinguin Solutions** – a freelance software development business based in Groningen, Netherlands.

I specialize in **Elixir** development (from embedded devices with Nerves to scalable Phoenix applications) and have extensive experience with **React**, **React Native**, and **AWS** cloud infrastructure.

### What I Offer

🚀 **Full-Stack Development**: From embedded IoT solutions to modern web applications
☁️ **Cloud Architecture**: AWS, Terraform, and infrastructure automation
🔒 **Security Operations**: Cloud security and compliance
⚡ **Performance Optimization**: Making your applications faster and more reliable

### Latest Insights

Here you'll find articles about real-world development challenges and their solutions. Whether you're dealing with Phoenix performance issues, React state management problems, or AWS security configurations, I share the solutions I've discovered through hands-on experience.

{% assign recent_posts = site.posts | slice: 0, 3 %}
{% for post in recent_posts %}

### [{{ post.title }}]({{ post.url }})

_{{ post.date | date: "%B %d, %Y" }}_

{{ post.excerpt | strip_html | truncate: 150 }}

[Read more →]({{ post.url }})
{% endfor %}

### Let's Work Together

Based in Groningen, I'm particularly interested in helping businesses in the Netherlands and surrounding regions. Whether you need help with a specific technical challenge or want to explore new technologies for your project, I'd love to hear from you.

[Get in touch](mailto:office@tonnenpinguin.solutions) or [learn more about my services](/about/).

---

_Follow me on [GitHub](https://github.com/tonnenpinguin/) and [LinkedIn](https://www.linkedin.com/in/bernhard-hackl-09a443b0/) for more technical insights._
