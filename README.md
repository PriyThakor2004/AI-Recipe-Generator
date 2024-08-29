# Import necessary modules
from flask import Flask, render_template_string, request
from boltiotai import openai
import os

# Set the OpenAI API key
openai.api_key = os.environ['OPENAI_API_KEY']

# Initialize Flask application
app = Flask(__name__)

# Function to generate a tutorial based on components
def generate_tutorial(components):
    try:
        response = openai.chat.completions.create(
            model="gpt-3.5-turbo",
            messages=[
                {"role": "system", "content": "You are a helpful assistant"},
                {
                    "role": "user",
                    "content": f"Suggest a recipe using the items listed as available. Make sure you have a nice name for this recipe listed at the start. Also, include a funny version of the name of the recipe on the following line. Then share the recipe in a step-by-step manner. In the end, write a fun fact about the recipe or any of the items used in the recipe. Here are the items available: {components}, Haldi, Chilly Powder, Tomato Ketchup, Water, Garam Masala, Oil"
                }
            ]
        )
        return response['choices'][0]['message']['content']
    except Exception as e:
        return f"An error occurred: {str(e)}"

# Define the root route for the Flask application
@app.route('/', methods=['GET', 'POST'])
def home():
    output = ""
    if request.method == 'POST':
        components = request.form['components']
        output = generate_tutorial(components)

    # HTML template for rendering the web page
    return render_template_string('''
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Custom Recipe Tutorial Generator</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0-alpha1/dist/css/bootstrap.min.css" rel="stylesheet"/>
    <style>
        body {
            background: linear-gradient(135deg, #1f1c2c, #928dab);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: #e0e0e0;
            overflow-x: hidden;
        }
        .background-animation {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: -1;
            background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, rgba(0,0,0,0) 80%);
            background-size: 200% 200%;
            animation: gradientAnimation 15s ease infinite;
        }
        @keyframes gradientAnimation {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }
        .container {
            max-width: 800px;
            margin-top: 50px;
            padding: 20px;
            background: rgba(0, 0, 0, 0.7);
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            backdrop-filter: blur(10px);
        }
        h1 {
            font-weight: 700;
            font-size: 2.8rem;
            color: #00ffe0;
            text-shadow: 0 0 10px rgba(0, 255, 224, 0.6);
        }
        .form-label {
            font-size: 1.1rem;
            color: #b0bec5;
        }
        .form-control {
            border: none;
            border-radius: 5px;
            background: rgba(255, 255, 255, 0.1);
            color: #ffffff;
            padding: 12px;
            transition: border 0.3s ease;
        }
        .form-control:focus {
            border: 1px solid #00ffe0;
            outline: none;
            box-shadow: 0 0 10px #00ffe0;
        }
        .btn-primary {
            background-color: #00ffe0;
            border: none;
            padding: 10px 20px;
            font-size: 1.1rem;
            transition: transform 0.3s, box-shadow 0.3s;
        }
        .btn-primary:hover {
            transform: translateY(-3px);
            box-shadow: 0 10px 20px rgba(0, 255, 224, 0.5);
        }
        .btn-secondary {
            background-color: #546e7a;
            border: none;
            transition: transform 0.3s, box-shadow 0.3s;
        }
        .btn-secondary:hover {
            transform: translateY(-3px);
            box-shadow: 0 10px 20px rgba(84, 110, 122, 0.5);
        }
        .card {
            border-radius: 12px;
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.2);
        }
        .card-header {
            background-color: rgba(0, 0, 0, 0.8);
            color: #00ffe0;
            font-size: 1.2rem;
            border-bottom: 1px solid #00ffe0;
        }
        .output-area {
            padding: 15px;
            border: 1px solid #00ffe0;
            border-radius: 8px;
            background-color: #263238;
            color: #cfd8dc;
            overflow-x: auto;
            font-family: monospace;
            animation: fadeIn 0.5s ease-in-out;
        }
        @keyframes fadeIn {
            from { opacity: 0; }
            to { opacity: 1; }
        }
    </style>
    <script>
        async function generateTutorial() {
            const components = document.querySelector("#components").value;
            const output = document.querySelector("#output");
            output.textContent = "Generating recipe...";
            try {
                const response = await fetch("/generate", {
                    method: "POST",
                    body: new FormData(document.querySelector("#tutorial-form")),
                });
                const newOutput = await response.text();
                output.textContent = newOutput;
            } catch (error) {
                output.textContent = "An error occurred: " + error.message;
            }
        }

        function copyToClipboard() {
            const output = document.querySelector("#output");
            const textarea = document.createElement("textarea");
            textarea.value = output.textContent;
            document.body.appendChild(textarea);
            textarea.select();
            document.execCommand("copy");
            document.body.removeChild(textarea);
            alert("Copied to clipboard");
        }
    </script>
</head>
<body>
    <div class="background-animation"></div>
    <div class="container">
        <header class="text-center mb-4">
            <h1>AI Recipe Generator By Priy</h1>
            <p class="lead">Enter the ingredients or items you have, and let us generate a recipe for you!</p>
        </header>

        <form id="tutorial-form" onsubmit="event.preventDefault(); generateTutorial();">
            <div class="mb-3">
                <label for="components" class="form-label">Ingredients / Items:</label>
                <input type="text" class="form-control" id="components" name="components" placeholder="Enter the list of ingredients or items you have (e.g., bread, jam, potato)" required />
            </div>
            <button type="submit" class="btn btn-primary">Generate Recipe</button>
        </form>

        <div class="card mt-4">
            <div class="card-header d-flex justify-content-between align-items-center">
                Recipe Output
                <button class="btn btn-secondary btn-sm" onclick="copyToClipboard()">Copy</button>
            </div>
            <div class="card-body output-area">
                <pre id="output" class="mb-0" style="white-space: pre-wrap;">{{ output }}</pre>
            </div>
        </div>
    </div>
</body>
</html>
''', output=output)

# Define the generate route to handle POST requests
@app.route('/generate', methods=['POST'])
def generate():
    components = request.form['components']
    return generate_tutorial(components)

# Run the Flask application
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
