# Zero-Shot Retail Checkout System

This project implements a real-time, zero-shot retail product identification and billing system using computer vision and vector search. It's designed to run on a local machine and uses a Streamlit application for the user interface.

## Architecture Overview

The system is composed of the following key components:

1.  **Streamlit Frontend (`app.py`):** A web-based interface for viewing the live camera feed, the current bill, and an admin panel for onboarding new products.

2.  **Real-time Video Processing (OpenCV):** Captures video from a webcam and performs basic motion detection to identify potential products in the checkout area.

3.  **Zero-Shot Product Recognition (CLIP & FAISS):**
    *   **CLIP Model:** A pre-trained vision-language model from Hugging Face (`openai/clip-vit-base-patch32`) is used to generate vector embeddings for images of products.
    *   **FAISS Index:** A vector database that stores the embeddings of all known products. When a new object is detected, its embedding is compared against the index to find the most similar product (zero-shot recognition).

4.  **Product Database (CSV & Local Files):**
    *   A `products.csv` file stores metadata for each product (SKU, name, price, etc.).
    *   Product images are stored in the `data/product_images` directory.

5.  **Low-Friction Onboarding Pipeline:**
    *   The Streamlit sidebar provides a simple form for an operator to add new products.
    *   The operator can upload images and enter product details.
    *   The system automatically generates embeddings for the new images and updates both the FAISS index and the product CSV file *without* restarting the main application.

## How It Works

1.  The application starts a video stream from the default webcam.
2.  A background subtraction algorithm detects moving objects in the frame, which are assumed to be products being placed in the checkout area.
3.  For each detected object, the system crops the object's image and generates a vector embedding using the CLIP model.
4.  This embedding is used to search the FAISS index for the nearest (most similar) product embedding.
5.  If a match is found with sufficient confidence, the product's information is retrieved from the product database.
6.  A simple tracking mechanism (based on SKU) adds the identified product to the bill. The UI updates in real-time to show the running tally and total cost.
7.  New products can be added on-the-fly via the admin panel, which seamlessly updates the knowledge base for future recognitions.

## How to Run the Application

1.  **Install Dependencies:**

    Make sure you have Python 3.8+ installed. Then, install the required packages from `requirements.txt`:

    ```bash
    pip install -r requirements.txt
    ```

2.  **Run the Streamlit App:**

    Open your terminal, navigate to the project directory, and run the following command:

    ```bash
    streamlit run app.py
    ```

    This will start the web server and open the application in your default browser.

3.  **Onboard Your First Product:**

    *   Before the camera feed can recognize anything, you need to add products to the system.
    *   Use the **Admin Panel** on the left sidebar.
    *   Fill in the SKU, Name, Category, and Price.
    *   Upload 1-3 clear images of the product.
    *   Click the "Add Product" button.
    *   Repeat for all the products you want the system to recognize.

4.  **Start Checkout:**

    *   Once you have onboarded some products, present them one by one to the camera.
    *   You should see a green box appear around the moving object, and the identified product's name and price will be displayed.
    *   The "Billing & Tally" section on the right will update with the identified products.

## Limitations & Future Improvements

*   **Object Detection:** The current system uses a simple background subtractor for motion detection. This is not robust and would be replaced with a proper object detection model (e.g., YOLO, Faster R-CNN) in a production environment.
*   **Object Tracking:** The tracking logic is very basic and does not handle occlusion or multiple instances of the same object well. A more advanced tracker like SORT or DeepSORT would be necessary for accurate counting.
*   **Confidence Threshold:** The `match_distance` threshold in the code is a magic number and would need to be carefully tuned for better accuracy.
*   **Scalability:** While FAISS is very fast, for a massive number of products, a more scalable vector database solution (e.g., Milvus, Weaviate) might be considered.
