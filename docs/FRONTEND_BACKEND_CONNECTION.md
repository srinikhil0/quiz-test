Frontend-Backend Connection Documentation

File Structure

QuizMaster/
├── app/                     
│   ├── services/           
│   │   └── api.ts         
│   ├── hooks/             
│   │   └── useQuiz.ts    
│   └── page.tsx            
├── api/                   
│   ├── queryLLM.js       
│   └── jsonGPT.js        


Request-Response Cycle
1. Frontend Request:

// app/services/api.ts
export const quizAPI = {
    submitQuestion: async (question: string): Promise<QuizResponse<LLMResponse>> => {
        return fetchAPI<QuizResponse<LLMResponse>>('/query', {
            method: 'POST',
            body: JSON.stringify({ prompt: question }),
        });
    }
};


2. Backend Processing:

// api/queryLLM.js
const queryLLM = async (req, res) => {
    const prompt = req.body.prompt;
    const answer = await jsonGPT(prompt);
    res.json({
        success: true,
        data: answer
    });
};


Component Breakdown

1. Frontend Components

API Service Layer (app/services/api.ts)

// Type definitions
export interface LLMResponse {
    questions: {
        question: string;
        options: string[];
        answer: number;
    }[];
}

export interface QuizResponse<T = LLMResponse> {
    success: boolean;
    data: T;
    message?: string;
}

// API calls
export const quizAPI = {
    submitQuestion: async (question: string) => {
        // API call implementation
    }
};


Custom Hook (app/hooks/useQuiz.ts)

export const useQuiz = () => {
    const [loading, setLoading] = useState(false);
    const [error, setError] = useState<string | null>(null);

    const submitQuestion = async (question: string) => {
        setLoading(true);
        try {
            const response = await quizAPI.submitQuestion(question);
            return response;
        } catch (err) {
            setError(err instanceof Error ? err.message : 'An error occurred');
            return null;
        } finally {
            setLoading(false);
        }
    };

    return { loading, error, submitQuestion };
};


2. Backend Components

API Endpoint (api/queryLLM.js)

const queryLLM = async (req, res) => {
    try {
        const prompt = req.body.prompt;
        const answer = await jsonGPT(prompt);
        res.json({ success: true, data: answer });
    } catch (error) {
        res.status(500).json({
            error: true,
            message: "Failed to process the request"
        });
    }
};


AI Service (api/jsonGPT.js)

const jsonGPT = async (prompt) => {
    const model = genAI.getGenerativeModel({ model: "gemini-pro" });
    const result = await model.generateContent(instructions);
    const response = await result.response;
    return JSON.parse(response.text());
};


API Integration

Endpoint Specifications

POST /query
- Purpose: Generate quiz questions from input text
- Request Body:
  json
  {
    "prompt": "Text to generate questions from"
  }
  
- Response:
  json
  {
    "success": true,
    "data": {
      "questions": [
        {
          "question": "Question text",
          "options": ["Option 1", "Option 2", "Option 3", "Option 4"],
          "answer": 1
        }
      ]
    }
  }
  

Error Handling

1. Frontend Error Handling
typescript
try {
    const result = await submitQuestion(inputText);
    if (result?.success) {
        setQuizData(result.data);
    } else {
        // Handle error state
    }
} catch (error) {
    console.error('Error:', error);
}


2. Backend Error Handling
javascript
try {
    // Process request
} catch (error) {
    res.status(500).json({
        error: true,
        message: "Failed to process the request",
        details: error.message
    });
}


Type Definitions

Frontend Types

interface Question {
    question: string;
    options: string[];
    answer: number;
}

interface QuizData {
    questions: Question[];
}

interface APIResponse<T> {
    success: boolean;
    data: T;
    message?: string;
}
