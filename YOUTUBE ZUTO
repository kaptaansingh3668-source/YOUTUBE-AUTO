import streamlit as st
import os
import subprocess
import whisper
import json
import requests

st.set_page_config(page_title="CreatorPilot AI", page_icon="🎬", layout="wide")

st.title("🎬 CreatorPilot AI")
st.caption("Your Free YouTube Automation Studio")

st.sidebar.title("Navigation")
page = st.sidebar.radio(
    "Select Feature",
    ["Dashboard", "Video to Shorts", "AI SEO", "YouTube Upload"]
)

os.makedirs("outputs", exist_ok=True)

if page == "Dashboard":
    st.header("Creator Dashboard")
    c1, c2, c3 = st.columns(3)
    c1.metric("Automation", "Active")
    c2.metric("Budget", "₹0")
    c3.metric("Platform", "YouTube")
    st.info("Upload your gameplay video to start creating Shorts.")

elif page == "Video to Shorts":
    st.header("Long Video → Shorts")
    video = st.file_uploader(
        "Upload your video",
        type=["mp4", "mov", "mkv"]
    )
    if video and st.button("Create Shorts"):
        with open("input.mp4", "wb") as f:
            f.write(video.read())

        with st.spinner("AI is analysing your video..."):
            try:
                model = whisper.load_model("base")
                result = model.transcribe("input.mp4", fp16=False)

                segments = result["segments"]
                if not segments:
                    st.warning("Speech was not detected.")
                else:
                    # Select non-overlapping speech moments.
                    selected = []
                    for seg in segments:
                        start = max(0, seg["start"] - 5)
                        end = min(seg["end"] + 20, start + 35)

                        if end - start < 10:
                            continue
                        if all(end <= x[1] or start >= x[2]
                               for x in selected):
                            selected.append(
                                (seg["text"], start, end)
                            )
                        if len(selected) == 5:
                            break

                    for i, (text, start, end) in enumerate(selected, 1):
                        output = f"outputs/short_{i}.mp4"
                        subprocess.run([
                            "ffmpeg", "-y",
                            "-ss", str(start),
                            "-i", "input.mp4",
                            "-t", str(end-start),
                            "-vf",
                            "scale=1080:1920:force_original_aspect_ratio=increase,crop=1080:1920",
                            "-c:v", "libx264",
                            "-c:a", "aac",
                            output
                        ], check=True, capture_output=True)

                        st.success(f"Short {i} created")
                        st.video(output)
                        st.caption(text)

            except Exception as e:
                st.error(str(e))

elif page == "AI SEO":
    st.header("YouTube SEO Generator")
    topic = st.text_input("Video topic", "GTA 6 gameplay")
    context = st.text_area("Video details")

    if st.button("Generate SEO"):
        prompt = f"""
        Create YouTube SEO metadata for this gaming video.
        Topic: {topic}
        Details: {context}
        Return JSON with title, description, tags and hashtags.
        """
        try:
            response = requests.post(
                "http://localhost:11434/api/generate",
                json={
                    "model": "llama3.2",
                    "prompt": prompt,
                    "stream": False
                },
                timeout=180
            )
            response.raise_for_status()
            raw = response.json()["response"]
            st.text_area("AI SEO Result", raw, height=300)
        except Exception:
            st.error("Install and run Ollama with llama3.2 first.")

elif page == "YouTube Upload":
    st.header("YouTube Upload")
    st.warning("YouTube API OAuth setup is required.")
    st.write("Upload feature will be connected in the next module.")
