python
import streamlit as st
import replicate
import os
import time
from dotenv import load_dotenv

# Load environment variables from .env file
load_dotenv()

# Configure the Streamlit page
st.set_page_config(
    page_title="AI Video Generator",
    page_icon="🎬",
    layout="centered"
)

st.title("🎬 AI Video Generation App")
st.write("Enter a prompt below to generate a video using cutting-edge AI models.")

# Check for API token
if not os.environ.get("REPLICATE_API_TOKEN"):
    st.warning("⚠️ Replicate API Token not found. Please set it in your .env file or sidebar.")
    api_token_input = st.sidebar.text_input("Enter Replicate API Token:", type="password")
    if api_token_input:
        os.environ["REPLICATE_API_TOKEN"] = api_token_input
        st.rerun()

# User Input Interface
prompt = st.text_area(
    "Video Prompt:", 
    placeholder="A cinematic shot of a futuristic city at sunset, cyberpunk aesthetic, 4k..."
)

# Advanced Options Sidebar
st.sidebar.header("Generation Settings")
model_choice = st.sidebar.selectbox(
    "Select AI Model",
    ["Luma Dream Machine", "Minimax Video", "Stable Video Diffusion"]
)

aspect_ratio = st.sidebar.selectbox(
    "Aspect Ratio",
    ["16:9", "9:16", "1:1"]
)

# Generation Logic
if st.button("Generate Video ✨", type="primary"):
    if not prompt:
        st.error("Please enter a text prompt first!")
    elif not os.environ.get("REPLICATE_API_TOKEN"):
        st.error("Please provide a valid Replicate API token.")
    else:
        with st.spinner("🎬 Your AI video is rendering... This may take a minute."):
            try:
                # Map user friendly choices to Replicate model strings
                # (Using Luma's Ray model on Replicate as a standard example)
                model_string = "luma/ray" 
                
                if model_choice == "Minimax Video":
                    model_string = "minimax/video-01"
                elif model_choice == "Stable Video Diffusion":
                    model_string = "stability-ai/stable-video-diffusion"

                # Trigger the prediction on Replicate
                output = replicate.run(
                    model_string,
                    input={
                        "prompt": prompt,
                        "aspect_ratio": aspect_ratio
                    }
                )
                
                # Replicate usually returns a URL string or a list containing the video URL
                video_url = output[0] if isinstance(output, list) else output

                if video_url:
                    st.success("🎉 Video generated successfully!")
                    # Display the video in the app
                    st.video(video_url)
                    
                    # Provide a download button
                    st.download_button(
                        label="Download Video",
                        data=video_url,
                        file_name="ai_generated_video.mp4",
                        mime="video/mp4"
                    )
                else:
                    st.error("Failed to retrieve video URL from the model output.")

            except Exception as e:
                st.error(f"An error occurred during generation: {str(e)}")

# Instructions footer
st.markdown("---")
st.caption("Powered by Streamlit and Replicate API.")
