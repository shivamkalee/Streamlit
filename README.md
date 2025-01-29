import streamlit as st
import pandas as pd
from collections import defaultdict

# Function to process the transcript and calculate scores
def process_transcript(transcript_text):
    speaker_counts = defaultdict(int)
    students_scores = defaultdict(lambda: {"attendance": 5, "questions_asked": 0, "answers": 0, "total_score": 5})
    teacher_feedback = ["good question", "good answer"]
    
    lines = transcript_text.split('\n')
    dialogues = []
    
    # Count the number of lines spoken by each speaker
    for line in lines:
        if '-' in line:
            parts = line.split('-')
            if len(parts) >= 2:
                speaker_info = parts[1].strip()
                speaker_name = speaker_info.split(' ')[0].strip()
                speaker_counts[speaker_name] += 1
                dialogues.append((speaker_name, line))
    
    # Identify the teacher as the person who spoke the most
    teacher = max(speaker_counts, key=speaker_counts.get)
    
    # Process dialogues for student participation
    for i, (speaker_name, line) in enumerate(dialogues):
        if speaker_name != teacher:
            parts = line.split('-')
            dialogue = ' '.join(parts[1:]).strip()
            if '?' in dialogue:  # Check for questions asked
                students_scores[speaker_name]["questions_asked"] += 1
            else:  # Assume the rest are answers
                students_scores[speaker_name]["answers"] += 1
                
            # Check if the next line contains teacher's feedback
            if i + 1 < len(dialogues):
                next_line_speaker = dialogues[i + 1][0]
                next_line_dialogue = dialogues[i + 1][1].lower()
                if next_line_speaker == teacher and any(feedback in next_line_dialogue for feedback in teacher_feedback):
                    students_scores[speaker_name]["total_score"] += 2
    
    for student in students_scores:
        students_scores[student]["total_score"] += students_scores[student]["questions_asked"] * 3
        students_scores[student]["total_score"] += students_scores[student]["answers"] * 3
    
    return teacher, students_scores

# Streamlit app
st.title("Student Participation Dashboard")

uploaded_file = st.file_uploader("Upload Transcript Text File", type="txt")
if uploaded_file is not None:
    transcript_text = uploaded_file.read().decode("utf-8")
    teacher, students_scores = process_transcript(transcript_text)
    
    st.subheader(f"This meeting was led by: {teacher}")
    
    # Convert the results to a DataFrame for easy visualization
    df = pd.DataFrame(students_scores).T.reset_index().rename(columns={"index": "Student Name"})
    df = df.sort_values(by="total_score", ascending=False)
    
    st.subheader("Leaderboard")
    st.table(df)
    
    winner = df.iloc[0]["Student Name"]
    runner_up = df.iloc[1]["Student Name"]
    
    st.subheader("Winner")
    st.write(f"🎉 {winner}")
    
    st.subheader("Runner-Up")
    st.write(f"🥈 {runner_up}")
