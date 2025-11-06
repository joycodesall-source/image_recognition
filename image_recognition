from flask import Flask, request
from serpapi import GoogleSearch
import requests

app = Flask(__name__)

# 🔑 Replace with your API keys
SERP_API_KEY = "3b3bb699d52f93d9694b4c0aab497dc56e5e454694c0d9e1044a7a801d4b9952"  # SerpApi
IMGBB_API_KEY = "2702b04076bf1011baf373d4367e662b"   # Free: https://api.imgbb.com/

@app.route("/", methods=["GET", "POST"])
def index():
    html_form = """
    <h2>Upload Image to Find Product & Prices</h2>
    <form method="POST" enctype="multipart/form-data">
        <input type="file" name="image" accept=".jpg,.png" required>
        <input type="submit" value="Search">
    </form>
    """

    if request.method == "POST":
        file = request.files.get("image")
        if not file:
            return html_form + "<p style='color:red;'>No file uploaded.</p>"

        if not file.filename.lower().endswith((".jpg", ".png", ".jpeg")):
            return html_form + "<p style='color:red;'>Invalid file type.</p>"

        # Upload image to imgbb
        try:
            response = requests.post(
                "https://api.imgbb.com/1/upload",
                data={"key": IMGBB_API_KEY},
                files={"image": file.read()}
            ).json()
            image_url = response["data"]["url"]
        except Exception as e:
            return html_form + f"<p style='color:red;'>Image upload error: {e}</p>"

        # Send image URL to SerpApi
        params = {
            "engine": "google_lens",
            "api_key": SERP_API_KEY,
            "url": image_url
        }

        try:
            search = GoogleSearch(params)
            results = search.get_dict()
        except Exception as e:
            return html_form + f"<p style='color:red;'>SerpApi error: {e}</p>"

        # Extract top 3 visual matches
        visual_matches = results.get("visual_matches", [])[:3]
        if not visual_matches:
            return html_form + "<p>No products found. Try a clearer image.</p>"

        result_html = f"<h3>Top 3 Product Matches:</h3><img src='{image_url}' width='200'><br><br>"

        for item in visual_matches:
            title = item.get("title", "No title")
            link = item.get("link", "#")
            price = item.get("price", "N/A")

            # Convert price to ₹ roughly
            if price and price != "N/A":
                price_number = "".join(c for c in price if c.isdigit() or c == ".")
                if price_number:
                    price = f"₹{price_number}"

            result_html += f"<b>{title}</b><br>Price: {price}<br>"
            result_html += f"<a href='{link}' target='_blank'>View Product</a><br><br>"

        return html_form + result_html

    return html_form


if __name__ == "__main__":
    print("Server running at http://127.0.0.1:5000")
    app.run(debug=True)
