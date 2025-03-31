# Project-Tools-in-Data-Science
import os
import pandas as pd
import json
import requests

# Ensure you have the correct CSV file path
csv_file_path = "movie-reviews.csv"  # Update this if necessary

# Load the CSV file
try:
    reviews = pd.read_csv(csv_file_path)
    print(f"Loaded {len(reviews)} reviews from {csv_file_path}")
except FileNotFoundError:
    print(f"Error: File '{csv_file_path}' not found.")
    exit()

# Set API key using environment variables (Secure practice)
os.environ["OPENAI_API_KEY"] = "eyJhbGciOiJIUzI1NiJ9.eyJlbWFpbCI6IjIzZjMwMDIyNjRAZHMuc3R1ZHkuaWl0bS5hYy5pbiJ9.CzYChUNptp2s-5wcDd6P0NvXrkfq5FV1RmwdQcedAv8"  # Set your API key here

def get_sentiment(review, debug=False):
    api_key = os.getenv("OPENAI_API_KEY")
    if not api_key:
        print("Error: API key not found. Set OPENAI_API_KEY environment variable.")
        return

    url = "http://aiproxy.sanand.workers.dev/openai/v1/chat/completions"
    headers = {"Authorization": f"Bearer {api_key}"}

    payload = {
        "model": "gpt-4o-mini",
        "messages": [
            {"role": "system", "content": "Identify the sentiment of the movie. JUST say positive / negative"},
            {"role": "user", "content": review}
        ]
    }

    try:
        response = requests.post(url, headers=headers, json=payload)

        # Check for errors
        if response.status_code != 200:
            print(f"Error {response.status_code}: {response.text}")
            return "Error: API request failed"

        result = response.json()

        # Extract sentiment result
        if 'choices' in result and result['choices']:
            answer = result["choices"][0].get("message", {}).get("content", "").strip()
            if debug:
                print(answer)
            return answer
        else:
            print(f"Unexpected response format: {result}")
            return "Error: Could not determine sentiment"

    except requests.exceptions.RequestException as e:
        print(f"Request failed: {e}")
        return "Error: Request failed"

# Example usage
if "review" in reviews.columns:
    get_sentiment(reviews.review.iloc[3], debug=True)
else:
    print("Error: CSV does not contain a 'review' column.")
