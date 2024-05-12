# Sending New Blog Post Notifications via Email: A Deep Dive into the Python Script
 

In the digital age, staying connected with your audience is crucial for maintaining engagement and driving traffic to your content. One effective way to keep your audience informed about new blog posts is through email notifications. In this article, we'll explore a Python script that automates the process of sending email notifications to subscribers whenever a new blog post is published.

**Understanding the Script**

Let's break down the script into its components and understand each line:

```python
import requests
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
import urllib
import os
```

- These lines import necessary modules for making HTTP requests (`requests`), sending emails (`smtplib`), and creating email message objects (`MIMEMultipart`, `MIMEText`, `MIMEImage`). `urllib` and `os` are imported for handling URLs and file operations, respectively.

```python
def send_mail(post_title, post_url, post_excerpt, to_address, image_url):
```

- This line defines the `send_mail` function, which takes parameters such as post title, URL, excerpt, recipient's email address, and image URL.

```python
    smtp_server = 'smtp.gmail.com'
    smtp_port = 587
    SENDER_USERNAME = 'your_mail@gmail.com'
    SENDER_PASSWORD = 'your_gmail_password'
```

- Here, we set the SMTP server and port for Gmail, along with the sender's Gmail address (`SENDER_USERNAME`) and password (`SENDER_PASSWORD`). Replace `'your_gmail_password'` with your actual Gmail account password or an app-specific password.

```python
    msg = MIMEMultipart('mixed')
    msg['Subject'] = "New Blog Post!"
    msg['From'] = "Deepdecide"
    msg['To'] = to_address
```

- These lines create a multipart email message (`msg`) with a subject line, sender name (`"Deepdecide"`), and recipient's email address (`to_address`). Note that we only specify the sender's name, not the email address, to ensure privacy.

```python
    img_data = urllib.request.urlopen(image_url).read()
    image = MIMEImage(img_data, name=os.path.basename(image_url))
    image.add_header('Content-ID', '<{}>'.format('featured'))
    msg.attach(image)
```

- This code fetches the featured image from the provided URL (`image_url`), creates a MIME image object, and attaches it to the email message. The image is embedded with a unique content ID (`'featured'`), allowing it to be referenced in the HTML content.

```python
    html = f"""
      <h1>{post_title}</h1>
      <p>{post_excerpt}</p>
      <a href="{post_url}" target="_blank">
        <h1>{post_title}<h1>
      </a>
      """
```

- Here, we construct the HTML content of the email, including the post title, excerpt, and a link to the full post. This HTML content is dynamically generated using string formatting (`f""`) and inserted into the email body.

```python
    part = MIMEText(html, 'html')
    msg.attach(part)
```

- This code creates a MIME text part containing the HTML content and attaches it to the email message.

```python
    server = smtplib.SMTP(smtp_server, smtp_port)
    server.starttls()
    server.login(SENDER_USERNAME, SENDER_PASSWORD)
```

- These lines establish a connection to the SMTP server (`smtp_server`) over TLS encryption (`starttls`) and authenticate the sender's Gmail account using the provided username and password (`SENDER_USERNAME` and `SENDER_PASSWORD`).

```python
    try:
        server.sendmail(SENDER_USERNAME, to_address, msg.as_string())
        print(f"Email sent to: {to_address}")
        return True
    except Exception as e:
        print(f"Failed to send email to: {to_address}. Error: {str(e)}")
        return False
    finally:
        server.quit()
```

- This `try-except-finally` block attempts to send the email message using the `sendmail` method of the SMTP server. If successful, it prints a success message, otherwise, it prints an error message. Finally, it closes the SMTP connection using `quit()`.

```python
def send_new_post_alert():
```

- This line defines the `send_new_post_alert` function, which retrieves information about the newest blog post and sends email notifications to subscribers.

```python
    with open('emails.txt', 'r') as f:
        emails = f.read().splitlines()
```

- Here, the script reads a list of subscriber email addresses from a text file named `emails.txt` and stores them in the `emails` list.

```python
    successful_count = 0
    failed_count = 0
```

- These variables are initialized to track the number of successfully sent and failed email notifications.

```python
    for email in emails:
        if send_mail(post_title, post_url, post_excerpt, email, image_url):
            successful_count += 1
        else:
            failed_count += 1
```

- This loop iterates over each subscriber email address, calling the `send_mail` function to send email notifications. It increments the appropriate count based on whether the email sending was successful or failed.

```python
    print(f"Total emails sent successfully: {successful_count}")
    print(f"Total emails failed to send: {failed_count}")
```

- Finally, the script prints the total number of email notifications sent successfully and the total number of failures.

**Complete Code**

```python
import requests
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
import urllib
import os

def send_mail(post_title, post_url, post_excerpt, to_address, image_url):
    smtp_server = 'smtp.gmail.com'
    smtp_port = 587
    SENDER_USERNAME = 'deepdecide@gmail.com'  # Gmail address from which emails will be sent
    SENDER_PASSWORD = 'your_gmail_password'   # Gmail password or app-specific password

    msg = MIMEMultipart('mixed')
    msg['Subject'] = "New Blog Post!"
    msg['From'] = "Deepdecide"  # Sender name, not email address
    msg['To'] = to_address

    # Embed the featured image in the html
    img_data = urllib.request.urlopen(image_url).read()      # image_url of the featured image
    image = MIMEImage(img_data, name=os.path.basename(image_url))
    image.add_header('Content-ID', '<{}>'.format('featured'))
    msg.attach(image)

    # Construct email in HTML format
    html = f"""
      <h1>{post_title}</h1>
      <p>{post_excerpt}</p>
      <a href="{post_url}" target="_blank">
        <h1>{post_title}<h1>
      </a>
      """

    part = MIMEText(html, 'html')
    msg.attach(part)

    server = smtplib.SMTP(smtp_server, smtp_port)
    server.starttls()
    server.login(SENDER_USERNAME, SENDER_PASSWORD)
    
    try:
        server.sendmail(SENDER_USERNAME, to_address, msg.as_string())
        print(f"Email sent to: {to_address}")  # Print the email address to which the email was successfully sent
        return True
    except Exception as e:
        print(f"Failed to send email to: {to_address}. Error: {str(e)}")  # Print the failed email address and error
        return False
    finally:
        server.quit()

def send_new_post_alert():
    response = requests.get('https://deepdecide.com/wp-json/wp/v2/posts?_embed')
    posts = response.json()

    newest_post = posts[0]
    post_title = newest_post['title']['rendered']
    post_url = newest_post['link']
    post_excerpt = newest_post['excerpt']['rendered']
    image_url = newest_post['_embedded']['wp:featuredmedia'][0]['media_details']['sizes']['medium']['source_url']

    with open('emails.txt', 'r') as f:
        emails = f.read().splitlines()

    successful_count = 0
    failed_count = 0

    for email in emails:
        if send_mail(post_title, post_url, post_excerpt, email, image_url):
            successful_count += 1
        else:
            failed_count += 1

    print(f"Total emails sent successfully: {successful_count}")
    print(f"Total emails failed to send: {failed_count}")

if __name__ == "__main__":
    send_new_post_alert()

```
  
