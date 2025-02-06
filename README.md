# SKN07-3rd-3Team

# 📘 RAG-based English Learning Chatbot
 
## 👥 Team Introduction
| <img src="https://github.com/user-attachments/assets/b5a17c3c-8415-409b-ae90-0a931e677fc3" width="250" height="250"/> | <img src="https://github.com/user-attachments/assets/005f1a53-0700-420e-8c62-ae1555dd538b" width="250" height="260"/> | <img src="https://github.com/user-attachments/assets/3009a31a-d5ab-469a-bf39-39e6c7779efe" width="250" height="250"/>  | 
|:----------:|:----------:|:----------:|
| English Teacher | Korean Teacher | Social Studies Teacher | 
| Joohyeok Seo | Sungwon Dae | Jungyeon Yoon | 

<br>

---

## 📖 Project Overview
- Project Name: RAG-based English Learning Chatbot
- Project Description: The AI High School Equivalency Exam (GED) Learning Tutor is a service designed to assist learners preparing for the GED exam. It is based on seven years of past GED exam questions, from 2018 to 2024. Users can input their questions, and the AI provides correct answers along with explanations.
  This project utilizes Retrieval-Augmented Generation (RAG) technology to facilitate English learning through a Streamlit-based chatbot. Users can either ask general questions or solve randomly generated questions from the FAISS database.
  
- Project Necessity (Background):
  - Traditional GED study materials lack an interactive system that provides instant responses to individual questions.
  - Since this system is designed based solely on past exam questions, learners can systematically and efficiently prepare for the GED exam.
  - The project aims to provide a public learning opportunity for students facing economic or environmental challenges that make formal education difficult, contributing to reducing the education gap.

- Project Objectives:
  This project aims to help learners prepare for the GED exam more effectively using AI technology.
  - Develop a Q&A Service: Utilize ChatGPT API to generate AI-based responses to user queries, based on GED exam questions from 2018 to 2024.
  - Implement NLP and LLM (ChatGPT API): Apply text analysis and document comprehension techniques to improve the accuracy of responses.
  - Provide a User-Friendly Interface: Design an intuitive and accessible learning environment for students.
  - Enhance Learning Efficiency: Offer a real-time AI-powered response system to facilitate effective learning.
  - Deliver Personalized Learning: Analyze the user's weak areas and provide targeted questions, answers, and explanations to support their learning progress.

---

## 🚀 Key Features

1. **General Question Response**
   - Generates responses using OpenAI GPT-3.5 based on user-input questions.
2. **Random Quiz Mode**
   - Retrieves random English questions from the FAISS database, allowing users to practice.
   - Provides multiple-choice options and allows users to check the correct answer.

---

## 📂 Project Structure

<img src="https://github.com/user-attachments/assets/81339928-6c06-4bf1-8871-71630856ecac" alt="Project Structure" width="800px">

---

## 🔧 Technology Stack
<p align="center">
  <img src="https://img.shields.io/badge/github-181717?style=flat-square&logo=github&logoColor=white" width="150" height="45" />
  <img src="https://img.shields.io/badge/git-F05032?style=flat-square&logo=git&logoColor=white" width="150" height="45" />
  <img src="https://img.shields.io/badge/python-3776AB?style=flat-square&logo=python&logoColor=white" width="150" height="45" />
  <img src="https://img.shields.io/badge/streamlit-FF4B4B?style=flat-square&logo=streamlit&logoColor=white" width="150" height="45" />
</p>
<br>
<p align="center">
  <img src="https://img.shields.io/badge/jupyter-F37626?style=flat-square&logo=jupyter&logoColor=white" width="150" height="45" />
  <img src="https://img.shields.io/badge/openai-412991?style=flat-square&logo=openai&logoColor=white" width="150" height="45" />
  <img src="https://img.shields.io/badge/discord-5865F2?style=flat-square&logo=discord&logoColor=white" width="150" height="45" />
</p>



---

## 📑 Main Procedures

### ▶️ 1. Data Collection – Web Crawling (Korea Institute for Curriculum and Evaluation, KICE)
- Extracting Code Numbers for Each Year, Education Level, Session, and Subject from the Main Page
```python
code_dict = {}
tmp = 1
while True:   # Retrieve All Page Information
    url = f'https://www.kice.re.kr/boardCnts/list.do?type=default&page={tmp}&selLimitYearYn=Y&selStartYear=2018&C06=&boardID=1500211&C05=&C04=&C03=&searchType=S&C02=&C01='
    re = requests.get(url)
    soup = BeautifulSoup(re.text, 'html.parser')
    
    if soup.find('td').text.strip() == 'No Registered Posts Available.':
        break
        
    info = soup.find('tbody').find_all('tr')
    
    for i in info:
        code = i.find_all('td')[0].text
        year = i.find_all('td')[1].text
        edu = i.find_all('td')[2].text
        cnt = i.find_all('td')[3].text
        subject = i.find_all('td')[4].find('a')['title']
        # Select Desired Data Information
        if edu == 'High School Equivalency Level' and subject == 'English':
            code_dict[code] = f'{year}_{edu}_{cnt}_{subject}'
    
    tmp += 1
```
![image](https://github.com/user-attachments/assets/be19cd7c-60ca-4ed9-a431-04b07a2ace09)
- Access Detailed Pages Using Extracted Code Numbers and Save PDF Files
```python
for code in code_dict.keys():
    down_url = f'https://www.kice.re.kr/boardCnts/view.do?boardID=1500211&boardSeq={code}&lev=0&m=030305&searchType=S&statusYN=W&page=1&s=kice'
    down_re = requests.get(down_url)
    down_soup = BeautifulSoup(down_re.text)
    tmp_url = down_soup.find(class_='fieldBox').find('a')['href']
    pdf_url = 'https://www.kice.re.kr' + tmp_url
    file_path = f'./data/Correct Answers/{code_dict[code]}.pdf'
    
    response = requests.get(pdf_url)
    # Save Files Only When the Response Status Code is 200 (Success)
    if response.status_code == 200:
        with open(file_path, 'wb') as file:
            file.write(response.content)  # Save PDF Files
```

</br>

### ▶️ 2. Data Extraction
### 2-1) Extract High School English Questions

- Extract Questions from PDF and Arrange Left/Right Columns in the Correct Order
```python
def extract_text_from_pdf(pdf_path):
    with pdfplumber.open(pdf_path) as pdf:
        combined_text_list = []

        for page in pdf.pages:
            width, height = page.width, page.height

            # Left Column Questions (Process Left Side First for Each Page)
            left_bbox = (0, 0, width / 2, height)
            left_crop = page.within_bbox(left_bbox)
            left_text = left_crop.extract_text()
            if left_text:
                combined_text_list.append(clean_text(left_text))  # Organize and Append Data

            # Right Column Questions (Process Right Side After Each Page)
            right_bbox = (width / 2, 0, width, height)
            right_crop = page.within_bbox(right_bbox)
            right_text = right_crop.extract_text()
            if right_text:
                combined_text_list.append(clean_text(right_text))
```

- OCR Processing for Image-based PDFs (Left to Right per Page)
```python
def extract_text_from_image_pdf(pdf_path):
    images = convert_from_path(pdf_path, dpi=300)
    combined_text_list = []

    for img in images:
        width, height = img.size

        # Left Column Question OCR
        left_crop = img.crop((0, 0, width // 2, height))
        left_text = pytesseract.image_to_string(left_crop, lang="eng+kor", config="--psm 6")
        combined_text_list.append(clean_text(left_text))

        # Right Column Question OCR
        right_crop = img.crop((width // 2, 0, width, height))
        right_text = pytesseract.image_to_string(right_crop, lang="eng+kor", config="--psm 6")
        combined_text_list.append(clean_text(right_text))

    return "\n".join(combined_text_list)
```

### 2-2) Extract High School English Answers
- Extract English Answers from PDF (Including OCR Processing)
```python
def extract_english_answers_from_pdf(pdf_path):
    answers = {}

    # 1️⃣ Extract Text Directly from PDF
    with pdfplumber.open(pdf_path) as pdf:
        text = "\n".join([page.extract_text() for page in pdf.pages if page.extract_text()])

    # 2️⃣ Apply OCR (Use OCR if Text is Empty)
    if not text.strip():
        text = extract_text_from_image_pdf(pdf_path)

    # 3️⃣ Output Raw Text Extracted via OCR (For Debugging Purposes)
    print("\n📝 OCR EXTRACTED TEXT FROM PDF:", pdf_path)
    print(text[:1000])  # Output First 1,000 Characters Only

    # 4️⃣ Find Sections Containing "English Answer Table" or "Session 3: English"
    match = re.search(r"(?:English Answer Table|Session 3: English|English)([\s\S]+?)(?=\n\w+ Answer Table|\Z)", text)
    if match:
        english_answers_section = match.group(1).strip()
    else:
        print(f"⚠ English Answers Not Found: {pdf_path}")
        return None

    # 5️⃣ Extract Answer Patterns (Include Debugging Output)
    extracted_text = convert_korean_numbers(english_answers_section)
    print("\n🔍 EXTRACTED ENGLISH ANSWERS SECTION:")
    print(extracted_text[:500])  # Output First 500 Characters Only

    # 6️⃣ Extract Question Numbers & Answers
    answer_pattern = re.findall(r"(\d+)\s+([①②③④1-4])", extracted_text)

    # 🔥 Debugging: Output Extracted Answers
    print("\n🔎 Extracted Answers Dictionary:", answer_pattern)

    for q_num, ans in answer_pattern:
        answers[q_num] = ans

    return answers
```

### 2-3) Merge High School English Questions and Answers into a JSON File
- Rename Files to Match Question and Answer Filenames
```python
def clean_filename(filename):
    return filename.replace("_High_School_Answers.pdf", "_High_School_English.pdf")  # Match Answer Filenames with Question Filenames
```

- Modify Key Values in the Converted Answer Data
```python
answers_data_fixed = {clean_filename(k): v for k, v in answers_data.items()}\
```

- Match Questions with Answers
```python
merged_data = {}

for file_name, question_content in questions_data.items():
    matched_file = clean_filename(file_name)
    if matched_file in answers_data_fixed:  # Add Only If an Answer Exists
        merged_data[file_name] = {
            "questions": question_content,
            "answers": answers_data_fixed[matched_file]
        }
    else:
        print(f"⚠ Question Files Without Answers: {file_name}")
```

</br>

### ▶️ 3. Embedding
- OpenAI의 "text-embedding-ada-002" Convert Questions into Vectors Using a Model
```python
def get_embedding(text):
    response = openai.embeddings.create(
        input=text,
        model="text-embedding-ada-002"
    )
    return response.data[0].embedding  # Apply the Latest API Method

# Convert All Questions into Embeddings
question_embeddings = [get_embedding(q) for q in questions]
```

- Create FAISS Database
```python
embedding_dim = 1536  # Set Vector Dimensions
index = faiss.IndexFlatL2(embedding_dim)  # Create FAISS Index
question_vectors = np.array(question_embeddings).astype("float32")  # Convert Embedding Data to NumPy Array
index.add(question_vectors)  # FAISS 데이터베이스에 추가
```

- Search for Similar Questions Using FAISS
```python
def search_similar_questions(query, top_k=3):
    # Convert Input Question into a Vector
    query_vector = np.array(get_embedding(query)).astype("float32").reshape(1, -1)

    # Search for the Closest Question
    distances, indices = index.search(query_vector, top_k)

    # Display Results
    print("\n[Most Similar Questions]")
    for i in range(top_k):
        idx = indices[0][i]
        print(f"{i+1}. {questions[idx]} (거리: {distances[0][i]:.4f})")
```

- Save FAISS Index
```python
faiss.write_index(index, "faiss_index.bin")
```

</br>

### ▶️ 4. Improve Performance with RAG (Retrieval-Augmented Generation)
- OpenAI Embedding
```python
client = openai.OpenAI()

# Embedding Conversion Function
def get_embedding(text):
    response = client.embeddings.create(
        input=text,
        model="text-embedding-ada-002"
    )
    return response.data[0].embedding  # Apply Latest API Method
```
- Search for Similar Questions in FAISS
```python
def search_similar_questions(query, top_k=3):
    query_vector = np.array(get_embedding(query)).astype("float32").reshape(1, -1)
    distances, indices = index.search(query_vector, top_k)

    # Generate List of Similar Questions
    similar_questions = [questions[idx] for idx in indices[0]]

    print("\n[Most Similar Questions]")
    for i, question in enumerate(similar_questions):
        print(f"{i+1}. {question} (Similarity Distance: {distances[0][i]:.4f})")

    return similar_questions
```
- Connect to GPT-4 for Final RAG Response Generation
```python
def generate_response(query):
    similar_questions = search_similar_questions(query, top_k=3)     # Find Similar Questions
    context = "\n".join(similar_questions)     # Use Retrieved Questions as Prompt Context

    # Setting up GPT-4 Prompt
    prompt = f"""
    You are an AI assistant. Use the following context to answer the question.

    Context:
    {context}

    Question: {query}

    Answer:
    """

    # GPT-4 API Call (Latest Method)
    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "system", "content": "You are a helpful assistant."},
                  {"role": "user", "content": prompt}]
    )

    return response.choices[0].message.content
```

</br>

### ▶️ 5. Streamlit Implementation
- Loading RAG-Related Functions
```python
def load_rag_functions():
    with open(rag_application_path, "r", encoding="utf-8") as f:
        nb_content = nbformat.read(f, as_version=4)
    
    exporter = PythonExporter()
    source_code, _ = exporter.from_notebook_node(nb_content)
    
    exec(source_code, globals())

load_rag_functions()
```

- Load FAISS database and question data
```python
faiss_index_path = "faiss_index.bin"  # The FAISS index path
index = faiss.read_index(faiss_index_path)  # Reading a FAISS index

faiss_data_path = "faiss_data.pkl"  # FAISS data path
try:
    with open(faiss_data_path, "rb") as f:
        faiss_data = pickle.load(f)  # Load question data
except FileNotFoundError:
    faiss_data = None  # If the file is not found, the data will be None
```

- Fetching a random question and handling the Streamlit UI
```python
def get_random_question_from_faiss():
    if faiss_data is not None and len(faiss_data) > 0:
        retrieved_data = random.choice(faiss_data)  # Select a random question
        if isinstance(retrieved_data, dict) and "question" in retrieved_data:
            question = retrieved_data["question"]
            options = retrieved_data.get("options", [])
            correct_answer = retrieved_data.get("answer", "")
            
            # Apply underline formatting
            question = question.replace("effort", "<u>effort</u>")
            return question, options, correct_answer
    return None, None, None
```

- Streamlit UI layout
```python
st.title("📘 RAG-based English learning chatbot")  # Setting the webpage title

query_type = st.radio("Select search type", ["General Question", "Solve Random Problem"])  # Select question type from the user

if query_type == "General Question":
    query = st.text_input("Please enter your question:")  # Receive general question input.
    if st.button("Generate response"):  # When the response button is clicked
        if query:
            with st.spinner("The AI is generating a response..."):
                answer = generate_response(query)  # Generate response using the GPT model
            st.subheader("Response from GPT-3.5")
            st.markdown(answer, unsafe_allow_html=True)  # Display response

elif query_type == "Solve a random problem":
    if "current_question" not in st.session_state:
        st.session_state.current_question = None
        st.session_state.current_options = []
        st.session_state.correct_answer = None
        st.session_state.answered = False

    if st.button("Generate random problem"):  # When the "Generate Random Problem" button is clicked
        result = get_random_question_from_faiss()
        if result and all(result):  # If there is valid problem data
            st.session_state.current_question, st.session_state.current_options, st.session_state.correct_answer = result
            st.session_state.answered = False
    
    if st.session_state.current_question:
        st.subheader("📖 Random problem")
        st.markdown(st.session_state.current_question, unsafe_allow_html=True)

        selected_option = st.radio("Please select the correct answer:", st.session_state.current_options, index=None)  # Display options

        if st.button("Check the answer"):  # When the "Check Answer" button is clicked
            if selected_option is None:
                st.warning("⚠️ Please select the correct answer!")
            else:
                st.session_state.answered = True
                if selected_option == st.session_state.correct_answer:
                    st.success("✅ Correct answer!")
                else:
                    st.error(f"❌ Incorrect answer! The correct answer is: {st.session_state.correct_answer}")

        if st.session_state.answered:
            if st.button("Generate a new problem"):  # When the "Generate New Problem" button is clicked
                st.session_state.current_question = None
                st.session_state.current_options = []
                st.session_state.correct_answer = None
                st.session_state.answered = False
                st.rerun()  # Refresh the page
```
---

## 🎬 Performance results (test/demonstration page)

## **1. Solve fill-in-the-blank question**  

<br>  

<p align="center">
  <img src="https://github.com/user-attachments/assets/c39017ae-5cb1-4718-b53b-9f78072266b6" width="600px" height="470px">
</p>

<br>  

I have identified an issue where the text containing blanks is missing during the PDF conversion process. This has caused the chatbot to fail in providing the fill-in-the-blank questions properly.

<br>  

To resolve this, I manually added the missing blanks using the _ (underscore) symbol after converting to JSON, thereby restoring the question format.  

<br>  

<p align="center">
  <img src="https://github.com/user-attachments/assets/2e2d1116-64e3-4e0e-9c3b-919bede8010a" width="600px" height="470px">
</p>

<br>  

---

## **2. Solve the issue with text containing underscores**  

<br>  

<p align="center">
  <img src="https://github.com/user-attachments/assets/7737df6d-8643-40fe-ba37-f571d3a8df99" width="600px" height="430px">
</p>

<br>  

I have identified an issue where underlined text is missing during the PDF conversion process. This has resulted in the loss of important parts within the options or the passage.  

<br>  

To resolve this, I adjusted the JSON file by inserting <u>text</u> around the underlined text to preserve the original PDF formatting.

<br>  

<p align="center">
  <img src="https://github.com/user-attachments/assets/d7652367-aaf7-440b-aeb3-96a4f7612bb3" width="600px" height="430px">
</p>

<br>  

---

## **3. Solve the issue with debugging message output**  

<br>  

<p align="center">
  <img src="https://github.com/user-attachments/assets/cb1fee0a-83da-452f-9209-5b8a3f9e01f9" width="600px" height="430px">
</p>

<br>  

An issue occurred where unintended internal debugging messages were being displayed during the chatbot's response process. As a result, unnecessary DEBUG and INFO messages were exposed to users during the problem-solving process.  

<br>  

To resolve this issue, I adjusted the logging level in streamlit.py to prevent DEBUG and INFO messages from being displayed.  

<br>  

<p align="center">
  <img src="https://github.com/user-attachments/assets/5dd23d3c-7536-4851-89a5-96d6f1d1b92e" width="600px" height="550px">
</p>

<br>  


## 📌 Additional improvements needed
Although the chatbot has been improved to generate fill-in-the-blank questions correctly, the following additional improvements are needed for better performance
<br>
1. The issue where the chatbot responds in English even though it receives questions in Korean has occurred. Some responses are being output in English.
2. An issue has been observed where certain elements (instructions, passage, options) are not properly converted during the JSON transformation process, causing some parts to be missing.
3. An issue has occurred where elements such as images, tables, and graphs included in the PDF are either missing or distorted during the text conversion process.

---
 
## 💭A one-line retrospective.

Joohyeok Seo: It was a project where I experienced both the performance and limitations of the latest embedding technology using OpenAI's text-embedding-ada-002 model.
<br>
SungWon Dae: I realized that in order to fully process the data and derive the desired results, more complex manual work may be required than expected.
<br>
Jeongyeon Yoon: While using the model, I realized how important the data processing steps are and how fine adjustments can significantly impact the results.

---

For the Korean version of this documentation, please refer to [README_ko.md](README_ko.md).
