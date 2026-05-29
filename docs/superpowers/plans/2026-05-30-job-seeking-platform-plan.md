# 求职准备平台 — 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a deployable AI-powered job-seeking platform covering position market insights, resume tailoring, interview experience search, and mock interviews.

**Architecture:** FastAPI backend with LangGraph agent orchestration, React + Shadcn frontend, PostgreSQL + pgvector for relational and vector data, Redis for task queue and caching, Docker Compose deployment.

**Tech Stack:** Python 3.11+, FastAPI, LangGraph, SQLAlchemy, pgvector, Redis, APScheduler, React 18, TypeScript, Shadcn UI, Vite, Docker

---

## Phase 1: Project Scaffolding and Infrastructure

### Task 1.1: Initialize Project Structure

**Files:**
- Create: `backend/requirements.txt`
- Create: `backend/app/__init__.py`
- Create: `backend/app/main.py`
- Create: `backend/app/config.py`
- Create: `backend/app/database.py`
- Create: `docker-compose.yml`
- Create: `backend/Dockerfile`

- [ ] **Step 1: Create backend requirements.txt**

```txt
fastapi==0.111.0
uvicorn[standard]==0.30.1
sqlalchemy==2.0.30
asyncpg==0.29.0
psycopg2-binary==2.9.9
pgvector==0.3.0
alembic==1.13.1
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.9
pydantic==2.7.1
pydantic-settings==2.3.0
redis==5.0.7
apscheduler==3.10.4
httpx==0.27.0
beautifulsoup4==4.12.3
langgraph==0.2.0
langchain==0.2.0
langchain-openai==0.1.8
pyyaml==6.0.1
pytest==8.2.0
pytest-asyncio==0.23.7
httpx==0.27.0
```

- [ ] **Step 2: Create docker-compose.yml at project root**

```yaml
version: "3.9"

services:
  db:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_USER: jobapp
      POSTGRES_PASSWORD: jobapp_secret
      POSTGRES_DB: jobapp
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgresql+asyncpg://jobapp:jobapp_secret@db:5432/jobapp
      DATABASE_URL_SYNC: postgresql://jobapp:jobapp_secret@db:5432/jobapp
      REDIS_URL: redis://redis:6379/0
      SECRET_KEY: dev-secret-change-in-production
      LLM_PROVIDER: deepseek
      LLM_API_KEY: ${LLM_API_KEY}
      LLM_MODEL: deepseek-chat
    depends_on:
      - db
      - redis
    volumes:
      - ./backend:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

volumes:
  pgdata:
```

- [ ] **Step 3: Create backend config module**

```python
# backend/app/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str = "postgresql+asyncpg://jobapp:jobapp_secret@localhost:5432/jobapp"
    database_url_sync: str = "postgresql://jobapp:jobapp_secret@localhost:5432/jobapp"
    redis_url: str = "redis://localhost:6379/0"
    secret_key: str = "dev-secret-change-in-production"
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 60

    llm_provider: str = "deepseek"
    llm_api_key: str = ""
    llm_model: str = "deepseek-chat"
    llm_base_url: str = "https://api.deepseek.com/v1"

    class Config:
        env_file = ".env"

settings = Settings()
```

- [ ] **Step 4: Create database module**

```python
# backend/app/database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase
from app.config import settings

engine = create_async_engine(settings.database_url, echo=False)
async_session = async_sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

async def get_db():
    async with async_session() as session:
        yield session
```

- [ ] **Step 5: Create minimal FastAPI app**

```python
# backend/app/main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI(title="Job Seeking Platform API")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/api/health")
async def health():
    return {"status": "ok"}
```

- [ ] **Step 6: Create backend Dockerfile**

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

RUN apt-get update && apt-get install -y build-essential && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

- [ ] **Step 7: Verify infra starts**

Run: `cd backend && pip install -r requirements.txt`
Run: `docker-compose up -d db redis`
Run: `docker-compose ps`
Expected: db and redis services show "running"
Run: `curl http://localhost:8000/api/health`
Expected: `{"status":"ok"}`

- [ ] **Step 8: Commit**

```bash
git add backend/ docker-compose.yml
git commit -m "feat: project scaffolding with FastAPI, PostgreSQL, Redis"
```

---

### Task 1.2: Initialize Frontend Project

**Files:**
- Create: frontend project via Vite
- Create: `frontend/src/App.tsx`
- Create: `frontend/src/main.tsx`
- Create: `frontend/src/api/client.ts`

- [ ] **Step 1: Scaffold React + TypeScript project with Vite**

Run: `cd /c/Users/ZhuanZ（无密码）/Documents/job-seeking-platform && npm create vite@latest frontend -- --template react-ts`
Run: `cd frontend && npm install`

- [ ] **Step 2: Install frontend dependencies**

Run: `cd frontend && npm install react-router-dom axios @tanstack/react-query lucide-react recharts tailwindcss @tailwindcss/vite`

- [ ] **Step 3: Install Shadcn UI**

Run: `cd frontend && npx shadcn@latest init -d`
Run: `cd frontend && npx shadcn@latest add button input card tabs dialog textarea select avatar dropdown-menu badge`

- [ ] **Step 4: Create API client**

```typescript
// frontend/src/api/client.ts
import axios from "axios";

const api = axios.create({
  baseURL: "http://localhost:8000/api",
});

api.interceptors.request.use((config) => {
  const token = localStorage.getItem("token");
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem("token");
      window.location.href = "/auth";
    }
    return Promise.reject(error);
  },
);

export default api;
```

- [ ] **Step 5: Create App entry with routing shell**

```tsx
// frontend/src/App.tsx
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import Layout from "./components/Layout";
import Dashboard from "./pages/Dashboard";

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route element={<Layout />}>
          <Route path="/" element={<Dashboard />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

- [ ] **Step 6: Create Layout component shell**

```tsx
// frontend/src/components/Layout.tsx
import { Outlet, Link, useLocation } from "react-router-dom";

const navItems = [
  { path: "/", label: "仪表盘" },
  { path: "/positions/market", label: "岗位洞察" },
  { path: "/resumes", label: "简历包装" },
  { path: "/interviews/experiences", label: "面经知识库" },
  { path: "/interviews/mock", label: "模拟面试" },
];

export default function Layout() {
  const location = useLocation();

  return (
    <div className="min-h-screen bg-gray-50">
      <header className="bg-white border-b sticky top-0 z-50">
        <div className="max-w-7xl mx-auto px-4 h-14 flex items-center justify-between">
          <Link to="/" className="font-bold text-lg">
            JobReady
          </Link>
          <nav className="flex gap-1">
            {navItems.map((item) => (
              <Link
                key={item.path}
                to={item.path}
                className={`px-3 py-1.5 rounded text-sm ${location.pathname === item.path ? "bg-gray-100 font-medium" : "text-gray-600 hover:bg-gray-50"}`}
              >
                {item.label}
              </Link>
            ))}
          </nav>
        </div>
      </header>
      <main className="max-w-7xl mx-auto px-4 py-6">
        <Outlet />
      </main>
    </div>
  );
}
```

- [ ] **Step 7: Verify frontend runs**

Run: `cd frontend && npm run dev`
Expected: Vite dev server starts on http://localhost:5173

- [ ] **Step 8: Commit**

```bash
git add frontend/
git commit -m "feat: initialize React frontend with routing and Shadcn UI"
```

---

## Phase 2: Authentication and User System

### Task 2.1: User Database Models

**Files:**
- Create: `backend/app/models/__init__.py`
- Create: `backend/app/models/user.py`
- Create: `backend/alembic.ini`
- Create: `backend/alembic/env.py`
- Create: `backend/alembic/versions/001_create_users.py`

- [ ] **Step 1: Create user models**

```python
# backend/app/models/__init__.py
from app.models.user import User, UserProfile

# backend/app/models/user.py
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey, JSON, func
from sqlalchemy.orm import relationship
from app.database import Base

class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, autoincrement=True)
    email = Column(String(255), unique=True, nullable=False, index=True)
    password_hash = Column(String(255), nullable=False)
    created_at = Column(DateTime, server_default=func.now())

    profile = relationship("UserProfile", back_populates="user", uselist=False)

class UserProfile(Base):
    __tablename__ = "user_profiles"

    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(Integer, ForeignKey("users.id"), unique=True, nullable=False)
    target_position = Column(String(100))
    skills = Column(JSON, default=list)
    education = Column(String(50))
    experience_years = Column(Integer)

    user = relationship("User", back_populates="profile")
```

- [ ] **Step 2: Create Pydantic schemas**

```python
# backend/app/schemas/user.py
from pydantic import BaseModel, EmailStr
from typing import Optional, List
from datetime import datetime

class UserRegister(BaseModel):
    email: str
    password: str

class UserLogin(BaseModel):
    email: str
    password: str

class UserProfileUpdate(BaseModel):
    target_position: Optional[str] = None
    skills: Optional[List[str]] = None
    education: Optional[str] = None
    experience_years: Optional[int] = None

class UserProfileResponse(BaseModel):
    target_position: Optional[str]
    skills: List[str]
    education: Optional[str]
    experience_years: Optional[int]

    class Config:
        from_attributes = True

class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
```

- [ ] **Step 3: Set up Alembic**

Run: `cd backend && pip install alembic && alembic init alembic`

Edit `backend/alembic/env.py`:
```python
from app.database import Base
from app.models import User, UserProfile
target_metadata = Base.metadata

from app.config import settings
config.set_main_option("sqlalchemy.url", settings.database_url_sync)
```

Run: `cd backend && alembic revision --autogenerate -m "create users and profiles"`
Run: `cd backend && PYTHONPATH=. alembic upgrade head`

- [ ] **Step 4: Verify tables exist**

Run: `docker-compose exec db psql -U jobapp -d jobapp -c "\dt"`
Expected: shows `users` and `user_profiles` tables

- [ ] **Step 5: Commit**

```bash
git add backend/app/models/ backend/app/schemas/user.py backend/alembic/
git commit -m "feat: add User and UserProfile models with migrations"
```

---

### Task 2.2: Auth API Endpoints

**Files:**
- Create: `backend/app/api/__init__.py`
- Create: `backend/app/api/auth.py`
- Create: `backend/app/middleware/__init__.py`
- Create: `backend/app/middleware/auth.py`
- Create: `backend/tests/test_auth.py`

- [ ] **Step 1: Write failing test for register**

```python
# backend/tests/test_auth.py
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app
from app.database import Base, engine

@pytest.fixture(autouse=True)
async def setup_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

@pytest.mark.asyncio
async def test_register():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as client:
        response = await client.post("/api/auth/register", json={
            "email": "test@example.com",
            "password": "password123"
        })
    assert response.status_code == 200
    data = response.json()
    assert "access_token" in data
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd backend && python -m pytest tests/test_auth.py::test_register -v`
Expected: FAIL (404 or similar, endpoint not defined)

- [ ] **Step 3: Create auth utilities**

```python
# backend/app/middleware/auth.py
from datetime import datetime, timedelta
from jose import JWTError, jwt
from passlib.context import CryptContext
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.database import get_db
from app.models.user import User
from app.config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
security = HTTPBearer()

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)

def create_access_token(user_id: int) -> str:
    expire = datetime.utcnow() + timedelta(minutes=settings.access_token_expire_minutes)
    return jwt.encode({"sub": str(user_id), "exp": expire}, settings.secret_key, algorithm=settings.algorithm)

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
    db: AsyncSession = Depends(get_db),
) -> User:
    try:
        payload = jwt.decode(credentials.credentials, settings.secret_key, algorithms=[settings.algorithm])
        user_id = int(payload.get("sub"))
    except (JWTError, ValueError):
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")
    result = await db.execute(select(User).where(User.id == user_id))
    user = result.scalar_one_or_none()
    if not user:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED)
    return user
```

- [ ] **Step 4: Create auth API**

```python
# backend/app/api/auth.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.database import get_db
from app.models.user import User, UserProfile
from app.schemas.user import UserRegister, UserLogin, TokenResponse
from app.middleware.auth import hash_password, verify_password, create_access_token, get_current_user

router = APIRouter(prefix="/api/auth", tags=["auth"])

@router.post("/register", response_model=TokenResponse)
async def register(body: UserRegister, db: AsyncSession = Depends(get_db)):
    existing = await db.execute(select(User).where(User.email == body.email))
    if existing.scalar_one_or_none():
        raise HTTPException(status_code=400, detail="Email already registered")
    user = User(email=body.email, password_hash=hash_password(body.password))
    db.add(user)
    await db.flush()
    profile = UserProfile(user_id=user.id, skills=[], experience_years=0)
    db.add(profile)
    await db.commit()
    return TokenResponse(access_token=create_access_token(user.id))

@router.post("/login", response_model=TokenResponse)
async def login(body: UserLogin, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(User).where(User.email == body.email))
    user = result.scalar_one_or_none()
    if not user or not verify_password(body.password, user.password_hash):
        raise HTTPException(status_code=401, detail="Invalid credentials")
    return TokenResponse(access_token=create_access_token(user.id))
```

- [ ] **Step 5: Register auth router in main.py**

Edit `backend/app/main.py`:
```python
from app.api.auth import router as auth_router
app.include_router(auth_router)
```

- [ ] **Step 6: Run test to verify it passes**

Run: `cd backend && python -m pytest tests/test_auth.py::test_register -v`
Expected: PASS

- [ ] **Step 7: Add login test**

```python
# Add to backend/tests/test_auth.py
@pytest.mark.asyncio
async def test_login():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as client:
        await client.post("/api/auth/register", json={
            "email": "login@test.com", "password": "pass123"
        })
        response = await client.post("/api/auth/login", json={
            "email": "login@test.com", "password": "pass123"
        })
    assert response.status_code == 200
    assert "access_token" in response.json()

@pytest.mark.asyncio
async def test_login_wrong_password():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as client:
        await client.post("/api/auth/register", json={
            "email": "wrong@test.com", "password": "pass123"
        })
        response = await client.post("/api/auth/login", json={
            "email": "wrong@test.com", "password": "wrong"
        })
    assert response.status_code == 401
```

Run: `cd backend && python -m pytest tests/test_auth.py -v`
Expected: all 3 tests PASS

- [ ] **Step 8: Commit**

```bash
git add backend/app/api/auth.py backend/app/middleware/auth.py backend/tests/test_auth.py backend/app/main.py
git commit -m "feat: add user registration and login with JWT auth"
```

---

### Task 2.3: User Profile API

**Files:**
- Create: `backend/app/api/user.py`
- Create: `backend/tests/test_user.py`

- [ ] **Step 1: Write failing test for profile update**

```python
# backend/tests/test_user.py
import pytest
from httpx import AsyncClient, ASGITransport
from app.main import app
from app.database import Base, engine

@pytest.fixture(autouse=True)
async def setup_db():
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)

async def get_token(client, email="profile@test.com", pw="pass123"):
    await client.post("/api/auth/register", json={"email": email, "password": pw})
    resp = await client.post("/api/auth/login", json={"email": email, "password": pw})
    return resp.json()["access_token"]

@pytest.mark.asyncio
async def test_update_profile():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as client:
        token = await get_token(client)
        response = await client.put("/api/user/profile", json={
            "target_position": "后端开发工程师",
            "skills": ["Python", "MySQL", "Linux"],
            "education": "本科",
            "experience_years": 1,
        }, headers={"Authorization": f"Bearer {token}"})
    assert response.status_code == 200
    data = response.json()
    assert data["target_position"] == "后端开发工程师"
    assert "Python" in data["skills"]
```

- [ ] **Step 2: Run test to verify it fails**

Run: `cd backend && python -m pytest tests/test_user.py::test_update_profile -v`
Expected: FAIL

- [ ] **Step 3: Create user profile API**

```python
# backend/app/api/user.py
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.database import get_db
from app.models.user import User, UserProfile
from app.schemas.user import UserProfileUpdate, UserProfileResponse
from app.middleware.auth import get_current_user

router = APIRouter(prefix="/api/user", tags=["user"])

@router.get("/profile", response_model=UserProfileResponse)
async def get_profile(
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    result = await db.execute(select(UserProfile).where(UserProfile.user_id == user.id))
    profile = result.scalar_one_or_none()
    return profile or UserProfile(user_id=user.id, skills=[])

@router.put("/profile", response_model=UserProfileResponse)
async def update_profile(
    body: UserProfileUpdate,
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    result = await db.execute(select(UserProfile).where(UserProfile.user_id == user.id))
    profile = result.scalar_one_or_none()
    if not profile:
        profile = UserProfile(user_id=user.id)
        db.add(profile)
    for field, value in body.model_dump(exclude_unset=True).items():
        setattr(profile, field, value)
    await db.commit()
    await db.refresh(profile)
    return profile
```

- [ ] **Step 4: Register router and run tests**

Edit `backend/app/main.py`, add:
```python
from app.api.user import router as user_router
app.include_router(user_router)
```

Run: `cd backend && python -m pytest tests/test_user.py -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add backend/app/api/user.py backend/tests/test_user.py backend/app/main.py
git commit -m "feat: add user profile CRUD API"
```

---

## Phase 3: Frontend Auth and Layout

### Task 3.1: Auth Pages and Auth Context

**Files:**
- Create: `frontend/src/pages/Login.tsx`
- Create: `frontend/src/pages/Register.tsx`
- Create: `frontend/src/hooks/useAuth.tsx`
- Create: `frontend/src/components/ProtectedRoute.tsx`

- [ ] **Step 1: Create auth context**

```tsx
// frontend/src/hooks/useAuth.tsx
import { createContext, useContext, useState, useEffect, ReactNode } from "react";
import api from "../api/client";

interface AuthContextType {
  token: string | null;
  isAuthenticated: boolean;
  login: (email: string, password: string) => Promise<void>;
  register: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

const AuthContext = createContext<AuthContextType | null>(null);

export function AuthProvider({ children }: { children: ReactNode }) {
  const [token, setToken] = useState<string | null>(localStorage.getItem("token"));

  useEffect(() => {
    if (token) {
      localStorage.setItem("token", token);
    } else {
      localStorage.removeItem("token");
    }
  }, [token]);

  const login = async (email: string, password: string) => {
    const response = await api.post("/auth/login", { email, password });
    setToken(response.data.access_token);
  };

  const register = async (email: string, password: string) => {
    const response = await api.post("/auth/register", { email, password });
    setToken(response.data.access_token);
  };

  const logout = () => setToken(null);

  return (
    <AuthContext.Provider value={{ token, isAuthenticated: !!token, login, register, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be inside AuthProvider");
  return ctx;
}
```

- [ ] **Step 2: Create ProtectedRoute**

```tsx
// frontend/src/components/ProtectedRoute.tsx
import { Navigate } from "react-router-dom";
import { useAuth } from "../hooks/useAuth";

export default function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const { isAuthenticated } = useAuth();
  if (!isAuthenticated) return <Navigate to="/auth" />;
  return <>{children}</>;
}
```

- [ ] **Step 3: Create login page**

```tsx
// frontend/src/pages/Login.tsx
import { useState } from "react";
import { useNavigate, Link } from "react-router-dom";
import { useAuth } from "../hooks/useAuth";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const { login } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    try {
      await login(email, password);
      navigate("/");
    } catch {
      setError("邮箱或密码错误");
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <Card className="w-96">
        <CardHeader>
          <CardTitle className="text-center">登录 JobReady</CardTitle>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            <Input type="email" placeholder="邮箱" value={email} onChange={(e) => setEmail(e.target.value)} required />
            <Input type="password" placeholder="密码" value={password} onChange={(e) => setPassword(e.target.value)} required />
            {error && <p className="text-red-500 text-sm">{error}</p>}
            <Button type="submit" className="w-full">登录</Button>
          </form>
          <p className="text-center text-sm mt-4 text-gray-500">
            还没有账号？<Link to="/auth/register" className="text-blue-600">注册</Link>
          </p>
        </CardContent>
      </Card>
    </div>
  );
}
```

- [ ] **Step 4: Create register page (similar pattern, abbreviated)**

```tsx
// frontend/src/pages/Register.tsx
import { useState } from "react";
import { useNavigate, Link } from "react-router-dom";
import { useAuth } from "../hooks/useAuth";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";

export default function Register() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [error, setError] = useState("");
  const { register } = useAuth();
  const navigate = useNavigate();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setError("");
    try {
      await register(email, password);
      navigate("/positions/profile");
    } catch {
      setError("注册失败，请重试");
    }
  };

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <Card className="w-96">
        <CardHeader>
          <CardTitle className="text-center">注册 JobReady</CardTitle>
        </CardHeader>
        <CardContent>
          <form onSubmit={handleSubmit} className="space-y-4">
            <Input type="email" placeholder="邮箱" value={email} onChange={(e) => setEmail(e.target.value)} required />
            <Input type="password" placeholder="密码（至少6位）" value={password} onChange={(e) => setPassword(e.target.value)} required minLength={6} />
            {error && <p className="text-red-500 text-sm">{error}</p>}
            <Button type="submit" className="w-full">注册</Button>
          </form>
          <p className="text-center text-sm mt-4 text-gray-500">
            已有账号？<Link to="/auth/login" className="text-blue-600">登录</Link>
          </p>
        </CardContent>
      </Card>
    </div>
  );
}
```

- [ ] **Step 5: Update App.tsx with auth routes**

```tsx
// frontend/src/App.tsx
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import { AuthProvider } from "./hooks/useAuth";
import Layout from "./components/Layout";
import ProtectedRoute from "./components/ProtectedRoute";
import Login from "./pages/Login";
import Register from "./pages/Register";
import Dashboard from "./pages/Dashboard";

function App() {
  return (
    <AuthProvider>
      <BrowserRouter>
        <Routes>
          <Route path="/auth/login" element={<Login />} />
          <Route path="/auth/register" element={<Register />} />
          <Route path="/auth" element={<Navigate to="/auth/login" />} />
          <Route element={<ProtectedRoute><Layout /></ProtectedRoute>}>
            <Route path="/" element={<Dashboard />} />
          </Route>
        </Routes>
      </BrowserRouter>
    </AuthProvider>
  );
}
```

- [ ] **Step 6: Verify frontend auth flow**

Run: `cd frontend && npm run dev`
Manual check: Navigate to http://localhost:5173 → redirected to /auth/login → register → redirected to dashboard

- [ ] **Step 7: Commit**

```bash
git add frontend/src/pages/Login.tsx frontend/src/pages/Register.tsx frontend/src/hooks/useAuth.tsx frontend/src/components/ProtectedRoute.tsx frontend/src/App.tsx
git commit -m "feat: add auth pages, auth context, and protected routes"
```

---

## Phase 4: Data Pipeline — Crawling and JD Storage

### Task 4.1: Crawl Log and JD Models

**Files:**
- Create: `backend/app/models/jd.py`
- Create: `backend/app/models/crawl.py`
- Create: `backend/app/schemas/jd.py`
- Create: `backend/app/schemas/crawl.py`

- [ ] **Step 1: Create JD and crawl models**

```python
# backend/app/models/jd.py
from sqlalchemy import Column, Integer, String, DateTime, Text, JSON, func
from app.database import Base

class JDRaw(Base):
    __tablename__ = "jd_raw"

    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(200), nullable=False)
    company = Column(String(200), nullable=False)
    category = Column(String(50), nullable=False)
    raw_text = Column(Text, nullable=False)
    salary_range = Column(String(100))
    city = Column(String(50))
    source_platform = Column(String(50), nullable=False)
    source_url = Column(String(500), unique=True, nullable=False)
    url_hash = Column(String(64), unique=True, nullable=False, index=True)
    crawled_at = Column(DateTime, server_default=func.now())
    published_at = Column(DateTime)

class PositionInsight(Base):
    __tablename__ = "position_insights"

    id = Column(Integer, primary_key=True, autoincrement=True)
    jd_id = Column(Integer, nullable=False)
    skill_tags = Column(JSON, default=list)
    high_freq_keywords = Column(JSON, default=list)
    business_keywords = Column(JSON, default=list)
    category = Column(String(50))
    generated_at = Column(DateTime, server_default=func.now())
```

```python
# backend/app/models/crawl.py
from sqlalchemy import Column, Integer, String, DateTime, Text, Float, func
from app.database import Base

class CrawlLog(Base):
    __tablename__ = "crawl_logs"

    id = Column(Integer, primary_key=True, autoincrement=True)
    platform = Column(String(50), nullable=False)
    keyword = Column(String(100), nullable=False)
    total_count = Column(Integer, default=0)
    new_count = Column(Integer, default=0)
    status = Column(String(20), default="success")
    duration_ms = Column(Float)
    error_msg = Column(Text)
    crawled_at = Column(DateTime, server_default=func.now())
```

- [ ] **Step 2: Update models __init__.py and run migration**

```python
# backend/app/models/__init__.py
from app.models.user import User, UserProfile
from app.models.jd import JDRaw, PositionInsight
from app.models.crawl import CrawlLog
```

Run: `cd backend && alembic revision --autogenerate -m "add jd, insights, crawl_logs tables"`
Run: `cd backend && PYTHONPATH=. alembic upgrade head`

- [ ] **Step 3: Create schemas**

```python
# backend/app/schemas/jd.py
from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

class JDRawResponse(BaseModel):
    id: int
    title: str
    company: str
    category: str
    raw_text: str
    salary_range: Optional[str]
    city: Optional[str]
    source_platform: str
    source_url: str
    crawled_at: datetime

    class Config:
        from_attributes = True

class PositionInsightResponse(BaseModel):
    id: int
    jd_id: int
    skill_tags: List[str]
    high_freq_keywords: List[str]
    business_keywords: List[str]
    category: Optional[str]
    generated_at: datetime

    class Config:
        from_attributes = True

class CapabilityProfileRequest(BaseModel):
    position_category: str  # "后端开发" / "AI产品经理" / "算法工程师"
    user_skills: List[str]
    education: Optional[str]
    experience_years: int

class SkillGap(BaseModel):
    skill: str
    status: str  # "已具备" / "部分具备" / "完全缺失"
    priority: int  # 1 = 最高优先级

class LearningPathItem(BaseModel):
    topic: str
    target_level: str  # 面试标准
    resources: List[str]
    estimated_hours: int

class CapabilityProfileResponse(BaseModel):
    radar_data: List[dict]
    skill_gaps: List[SkillGap]
    learning_path: List[LearningPathItem]
```

```python
# backend/app/schemas/crawl.py
from pydantic import BaseModel
from datetime import datetime
from typing import Optional

class CrawlLogResponse(BaseModel):
    id: int
    platform: str
    keyword: str
    total_count: int
    new_count: int
    status: str
    duration_ms: Optional[float] = None
    crawled_at: datetime

    class Config:
        from_attributes = True
```

- [ ] **Step 4: Commit**

```bash
git add backend/app/models/jd.py backend/app/models/crawl.py backend/app/schemas/jd.py backend/app/schemas/crawl.py backend/alembic/
git commit -m "feat: add JD, position insights, and crawl log models"
```

---

### Task 4.2: Crawler Base and Scheduler

**Files:**
- Create: `backend/app/services/__init__.py`
- Create: `backend/app/services/crawler/__init__.py`
- Create: `backend/app/services/crawler/base.py`
- Create: `backend/app/services/crawler/scheduler.py`

- [ ] **Step 1: Create base crawler interface**

```python
# backend/app/services/crawler/base.py
from abc import ABC, abstractmethod
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime

@dataclass
class JDRawItem:
    title: str
    company: str
    category: str
    raw_text: str
    salary_range: Optional[str]
    city: Optional[str]
    source_platform: str
    source_url: str
    published_at: Optional[datetime]

class BaseCrawler(ABC):
    def __init__(self, keyword: str, category: str):
        self.keyword = keyword
        self.category = category

    @abstractmethod
    async def crawl(self) -> List[JDRawItem]:
        pass

    @abstractmethod
    def platform_name(self) -> str:
        pass
```

- [ ] **Step 2: Create crawler scheduler service**

```python
# backend/app/services/crawler/scheduler.py
import hashlib
import time
import traceback
import logging
from typing import List, Type
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.database import async_session
from app.models.jd import JDRaw
from app.models.crawl import CrawlLog
from app.services.crawler.base import BaseCrawler, JDRawItem

logger = logging.getLogger(__name__)

CRAWLER_REGISTRY: List[Type[BaseCrawler]] = []

def register_crawler(cls: Type[BaseCrawler]):
    CRAWLER_REGISTRY.append(cls)
    return cls

async def run_crawler_job(keyword: str, category: str):
    async with async_session() as db:
        for crawler_cls in CRAWLER_REGISTRY:
            crawler = crawler_cls(keyword=keyword, category=category)
            start = time.time()
            try:
                items = await crawler.crawl()
                new_count = 0
                for item in items:
                    url_hash = hashlib.sha256(item.source_url.encode()).hexdigest()
                    existing = await db.execute(
                        select(JDRaw).where(JDRaw.url_hash == url_hash)
                    )
                    if existing.scalar_one_or_none():
                        continue
                    jd = JDRaw(
                        title=item.title,
                        company=item.company,
                        category=item.category,
                        raw_text=item.raw_text,
                        salary_range=item.salary_range,
                        city=item.city,
                        source_platform=crawler.platform_name(),
                        source_url=item.source_url,
                        url_hash=url_hash,
                        published_at=item.published_at,
                    )
                    db.add(jd)
                    new_count += 1

                log = CrawlLog(
                    platform=crawler.platform_name(),
                    keyword=keyword,
                    total_count=len(items),
                    new_count=new_count,
                    status="success",
                    duration_ms=(time.time() - start) * 1000,
                )
                db.add(log)
                await db.commit()
            except Exception as e:
                log = CrawlLog(
                    platform=crawler.platform_name(),
                    keyword=keyword,
                    total_count=0,
                    new_count=0,
                    status="failed",
                    error_msg=traceback.format_exc()[:4000],
                    duration_ms=(time.time() - start) * 1000,
                )
                db.add(log)
                await db.commit()
                logger.error(f"Crawler {crawler.platform_name()} failed: {e}")
```

- [ ] **Step 3: Create scheduler startup/shutdown hook**

```python
# backend/app/services/crawler/scheduler.py (add to file)
from apscheduler.schedulers.asyncio import AsyncIOScheduler

scheduler = AsyncIOScheduler()

def start_scheduler():
    from app.services.crawler.scheduler import run_crawler_job
    keywords = [
        ("后端开发", "后端开发"),
        ("AI产品经理", "AI产品经理"),
        ("算法工程师", "算法工程师"),
    ]
    for keyword, category in keywords:
        scheduler.add_job(
            run_crawler_job,
            "interval",
            hours=6,
            args=[keyword, category],
            id=f"crawl_{keyword}",
            replace_existing=True,
        )
    scheduler.start()

def shutdown_scheduler():
    scheduler.shutdown(wait=False)
```

- [ ] **Step 4: Wire into FastAPI startup/shutdown**

```python
# backend/app/main.py (add startup/shutdown)
from app.services.crawler.scheduler import start_scheduler, shutdown_scheduler

app.add_event_handler("startup", start_scheduler)
app.add_event_handler("shutdown", shutdown_scheduler)
```

- [ ] **Step 5: Verify scheduler starts without errors**

Run: `cd backend && python -c "from app.main import app; print('startup OK')"`
Expected: `startup OK` (no import errors, scheduler starts)

- [ ] **Step 6: Commit**

```bash
git add backend/app/services/__init__.py backend/app/services/crawler/
git commit -m "feat: add crawler base interface and scheduler with APScheduler"
```

---

### Task 4.3: Boss Zhipin Crawler

**Files:**
- Create: `backend/app/services/crawler/boss_crawler.py`
- Create: `backend/tests/test_crawlers.py`

- [ ] **Step 1: Create Boss Zhipin crawler with placeholder**

```python
# backend/app/services/crawler/boss_crawler.py
import json
import re
from typing import List
from datetime import datetime
import httpx
from app.services.crawler.base import BaseCrawler, JDRawItem, register_crawler

BOSS_SEARCH_URL = "https://www.zhipin.com/wapi/zpgeek/search/joblist.json"

@register_crawler
class BossZhipinCrawler(BaseCrawler):
    async def crawl(self) -> List[JDRawItem]:
        items = []
        async with httpx.AsyncClient(timeout=30.0) as client:
            try:
                response = await client.get(
                    BOSS_SEARCH_URL,
                    params={"query": self.keyword, "page": 1, "pageSize": 30},
                    headers={
                        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36",
                        "Referer": "https://www.zhipin.com/",
                    },
                )
                if response.status_code != 200:
                    return items
                data = response.json()
                jobs = data.get("zpData", {}).get("jobList", [])
                for job in jobs:
                    raw_text = f"{job.get('jobName', '')}\n{job.get('jobDescription', '')}\n{job.get('skills', '')}"
                    if not raw_text.strip():
                        continue
                    items.append(JDRawItem(
                        title=job.get("jobName", ""),
                        company=job.get("brandName", ""),
                        category=self.category,
                        raw_text=raw_text,
                        salary_range=job.get("salaryDesc", ""),
                        city=job.get("cityName", ""),
                        source_platform=self.platform_name(),
                        source_url=f"https://www.zhipin.com/job_detail/{job.get('encryptJobId', '')}.html",
                        published_at=datetime.now(),
                    ))
            except Exception:
                pass
        return items

    def platform_name(self) -> str:
        return "BOSS直聘"
```

- [ ] **Step 2: Create crawler unit test**

```python
# backend/tests/test_crawlers.py
import pytest
from app.services.crawler.base import JDRawItem
from app.services.crawler.boss_crawler import BossZhipinCrawler

def test_boss_crawler_platform_name():
    crawler = BossZhipinCrawler(keyword="后端开发", category="后端开发")
    assert crawler.platform_name() == "BOSS直聘"

def test_jd_raw_item_creation():
    item = JDRawItem(
        title="后端开发工程师",
        company="测试公司",
        category="后端开发",
        raw_text="岗位职责：负责后端服务开发",
        salary_range="15-25K",
        city="北京",
        source_platform="BOSS直聘",
        source_url="https://example.com/job/1",
        published_at=None,
    )
    assert item.title == "后端开发工程师"
    assert item.company == "测试公司"
```

- [ ] **Step 3: Run tests**

Run: `cd backend && python -m pytest tests/test_crawlers.py -v`
Expected: 2 tests PASS

- [ ] **Step 4: Commit**

```bash
git add backend/app/services/crawler/boss_crawler.py backend/tests/test_crawlers.py
git commit -m "feat: add Boss Zhipin crawler implementation"
```

---

### Task 4.4: Crawl Logs API (Dashboard)

**Files:**
- Create: `backend/app/api/dashboard.py`

- [ ] **Step 1: Create dashboard API for recent crawl logs**

```python
# backend/app/api/dashboard.py
from datetime import datetime, timedelta
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func
from app.database import get_db
from app.models.crawl import CrawlLog
from app.models.jd import JDRaw
from app.schemas.crawl import CrawlLogResponse

router = APIRouter(prefix="/api/dashboard", tags=["dashboard"])

@router.get("/crawl-logs", response_model=list[CrawlLogResponse])
async def get_recent_crawl_logs(db: AsyncSession = Depends(get_db)):
    since = datetime.utcnow() - timedelta(days=1)
    result = await db.execute(
        select(CrawlLog)
        .where(CrawlLog.crawled_at >= since)
        .order_by(CrawlLog.crawled_at.desc())
        .limit(50)
    )
    return result.scalars().all()

@router.get("/stats")
async def get_dashboard_stats(db: AsyncSession = Depends(get_db)):
    since = datetime.utcnow() - timedelta(days=1)

    total_jd_result = await db.execute(select(func.count(JDRaw.id)))
    total_jd = total_jd_result.scalar()

    recent_jd_result = await db.execute(
        select(func.count(JDRaw.id)).where(JDRaw.crawled_at >= since)
    )
    recent_jd = recent_jd_result.scalar()

    recent_crawl_result = await db.execute(
        select(func.count(CrawlLog.id)).where(CrawlLog.crawled_at >= since)
    )
    recent_crawl = recent_crawl_result.scalar()

    return {
        "total_jd_count": total_jd,
        "recent_jd_count": recent_jd,
        "recent_crawl_count": recent_crawl,
    }
```

- [ ] **Step 2: Register in main.py**

```python
# backend/app/main.py (add)
from app.api.dashboard import router as dashboard_router
app.include_router(dashboard_router)
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/api/dashboard.py
git commit -m "feat: add dashboard API for crawl logs and stats"
```

---

## Phase 5: Position Insights — Market Overview and Capability Profile

### Task 5.1: JD Listing and Search API

**Files:**
- Create: `backend/app/api/positions.py`

- [ ] **Step 1: Create positions API with filtering**

```python
# backend/app/api/positions.py
from typing import Optional
from fastapi import APIRouter, Depends, Query
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, func
from app.database import get_db
from app.models.jd import JDRaw, PositionInsight
from app.schemas.jd import JDRawResponse

router = APIRouter(prefix="/api/positions", tags=["positions"])

@router.get("/jd", response_model=list[JDRawResponse])
async def list_jds(
    category: Optional[str] = Query(None),
    city: Optional[str] = Query(None),
    keyword: Optional[str] = Query(None),
    limit: int = Query(20, le=100),
    offset: int = Query(0),
    db: AsyncSession = Depends(get_db),
):
    q = select(JDRaw)
    if category:
        q = q.where(JDRaw.category == category)
    if city:
        q = q.where(JDRaw.city == city)
    if keyword:
        q = q.where(JDRaw.raw_text.ilike(f"%{keyword}%"))
    q = q.order_by(JDRaw.crawled_at.desc()).offset(offset).limit(limit)
    result = await db.execute(q)
    return result.scalars().all()

@router.get("/jd/{jd_id}", response_model=JDRawResponse)
async def get_jd_detail(jd_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(JDRaw).where(JDRaw.id == jd_id))
    return result.scalar_one()

@router.get("/categories")
async def get_categories(db: AsyncSession = Depends(get_db)):
    result = await db.execute(
        select(JDRaw.category, func.count(JDRaw.id))
        .group_by(JDRaw.category)
    )
    return [{"category": row[0], "count": row[1]} for row in result.all()]
```

- [ ] **Step 2: Register router and test**

Edit `backend/app/main.py`:
```python
from app.api.positions import router as positions_router
app.include_router(positions_router)
```

Run: `cd backend && curl http://localhost:8000/api/positions/jd?category=后端开发`
Expected: `[]` (empty list until crawlers fill data)

- [ ] **Step 3: Commit**

```bash
git add backend/app/api/positions.py
git commit -m "feat: add JD listing and search API with filtering"
```

---

## Phase 6: LLM Service and Agent Framework

### Task 6.1: LLM Service with Configuration-Driven Provider

**Files:**
- Create: `backend/app/services/llm.py`
- Create: `backend/app/services/agents/__init__.py`
- Create: `backend/app/services/agents/trace.py`

- [ ] **Step 1: Create LLM service**

```python
# backend/app/services/llm.py
from langchain_openai import ChatOpenAI
from app.config import settings
import json
import time
import logging

logger = logging.getLogger(__name__)

_llm_instance = None

def get_llm() -> ChatOpenAI:
    global _llm_instance
    if _llm_instance is None:
        _llm_instance = ChatOpenAI(
            model=settings.llm_model,
            api_key=settings.llm_api_key,
            base_url=settings.llm_base_url,
            temperature=0.3,
            max_tokens=settings.llm_model == "deepseek-chat" and 4096 or None,
        )
    return _llm_instance

async def call_llm(prompt: str, system_prompt: str = "") -> str:
    llm = get_llm()
    messages = []
    if system_prompt:
        messages.append({"role": "system", "content": system_prompt})
    messages.append({"role": "user", "content": prompt})
    start = time.time()
    try:
        response = await llm.ainvoke(messages)
        elapsed = time.time() - start
        logger.info(f"LLM call: {elapsed:.2f}s, {len(response.content)} chars")
        return response.content
    except Exception as e:
        elapsed = time.time() - start
        logger.error(f"LLM call failed after {elapsed:.2f}s: {e}")
        raise

async def call_llm_structured(prompt: str, system_prompt: str = "") -> dict:
    full_prompt = prompt + "\n\n请以 JSON 格式返回结果，不要包含任何其他文本。"
    text = await call_llm(full_prompt, system_prompt)
    text = text.strip()
    if text.startswith("```json"):
        text = text[7:]
    if text.startswith("```"):
        text = text[3:]
    if text.endswith("```"):
        text = text[:-3]
    return json.loads(text.strip())
```

- [ ] **Step 2: Create agent trace utility**

```python
# backend/app/services/agents/trace.py
import time
import json
from typing import Any

class AgentTrace:
    def __init__(self, agent_type: str, input_summary: str, db_session=None):
        self.agent_type = agent_type
        self.input_summary = input_summary[:500]
        self.steps = []
        self.db = db_session
        self.start = time.time()

    def record_step(self, name: str, output: Any = None):
        self.steps.append({"name": name, "output": str(output)[:500]})

    async def save(self, status: str = "success", error_msg: str = None):
        from app.models.crawl import CrawlLog  # reuse pattern
        elapsed = (time.time() - self.start) * 1000
        if not self.db:
            return
        # Simplified: just log trace
        import logging
        logging.info(f"AgentTrace[{self.agent_type}]: {status} in {elapsed:.0f}ms, {len(self.steps)} steps")
```

- [ ] **Step 3: Commit**

```bash
git add backend/app/services/llm.py backend/app/services/agents/
git commit -m "feat: add LLM service with config-driven provider and agent tracing"
```

---

### Task 6.2: JD Analysis Service (Single LLM Call, Not Agent)

**Files:**
- Create: `backend/app/services/jd_analyzer.py`

- [ ] **Step 1: Create JD analysis service**

```python
# backend/app/services/jd_analyzer.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.models.jd import JDRaw, PositionInsight
from app.services.llm import call_llm_structured

async def analyze_jd(jd_id: int, db: AsyncSession) -> PositionInsight:
    result = await db.execute(select(JDRaw).where(JDRaw.id == jd_id))
    jd = result.scalar_one()

    prompt = f"""
分析以下招聘 JD，提取关键信息：

岗位：{jd.title}
公司：{jd.company}
JD 内容：
{jd.raw_text[:3000]}

请提取：
1. skill_tags: 硬技能标签列表（编程语言、框架、工具等）
2. business_keywords: 业务领域关键词（电商、金融、广告等）
3. category: 岗位分类
"""

    data = await call_llm_structured(prompt)
    insight = PositionInsight(
        jd_id=jd_id,
        skill_tags=data.get("skill_tags", []),
        high_freq_keywords=data.get("skill_tags", [])[:10],
        business_keywords=data.get("business_keywords", []),
        category=data.get("category", jd.category),
    )
    db.add(insight)
    await db.commit()
    return insight

async def batch_analyze_jds(db: AsyncSession):
    result = await db.execute(
        select(JDRaw).outerjoin(
            PositionInsight, JDRaw.id == PositionInsight.jd_id
        ).where(PositionInsight.id == None).limit(20)
    )
    unanalyzed = result.scalars().all()
    for jd in unanalyzed:
        try:
            await analyze_jd(jd.id, db)
        except Exception as e:
            import logging
            logging.error(f"Failed to analyze JD {jd.id}: {e}")
```

- [ ] **Step 2: Commit**

```bash
git add backend/app/services/jd_analyzer.py
git commit -m "feat: add JD analysis service with LLM-based skill extraction"
```

---

## Phase 7: Capability Profile Agent (Agent 1)

### Task 7.1: Capability Profile Agent with LangGraph

**Files:**
- Create: `backend/app/services/agents/profile_agent.py`
- Create: `backend/app/api/skills.py`

- [ ] **Step 1: Create capability profile agent**

```python
# backend/app/services/agents/profile_agent.py
from typing import TypedDict, List
from langgraph.graph import StateGraph, END
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.models.jd import JDRaw, PositionInsight
from app.models.user import UserProfile
from app.services.llm import call_llm, call_llm_structured

class ProfileState(TypedDict):
    position_category: str
    user_skills: List[str]
    education: str
    experience_years: int
    common_skills: List[str]
    skill_gaps: List[dict]
    learning_path: List[dict]
    radar_data: List[dict]

async def step_market_scan(state: ProfileState, db: AsyncSession) -> ProfileState:
    result = await db.execute(
        select(PositionInsight)
        .where(PositionInsight.category == state["position_category"])
        .limit(50)
    )
    insights = result.scalars().all()

    all_skills = []
    for ins in insights:
        all_skills.extend(ins.skill_tags or [])

    prompt = f"""
以下是 {state['position_category']} 岗位的所有技能标签：
{all_skills[:200]}

请从中提炼出该岗位的通用核心能力模型，分为：
1. 必须掌握（出现频率最高的 5-8 项）
2. 加分项（有差异化价值的 3-5 项）

以 JSON 格式返回：
{{"must_have": ["skill1", "skill2"], "nice_to_have": ["skill3"]}}
"""
    data = await call_llm_structured(prompt)
    state["common_skills"] = data.get("must_have", []) + data.get("nice_to_have", [])
    return state

async def step_gap_analysis(state: ProfileState) -> ProfileState:
    user_skills_lower = [s.lower() for s in state["user_skills"]]
    gaps = []
    for skill in state["common_skills"]:
        if skill.lower() in user_skills_lower:
            gaps.append({"skill": skill, "status": "已具备", "priority": 3})
        elif any(skill.lower() in us for us in user_skills_lower):
            gaps.append({"skill": skill, "status": "部分具备", "priority": 2})
        else:
            gaps.append({"skill": skill, "status": "完全缺失", "priority": 1})
    gaps.sort(key=lambda g: g["priority"])
    state["skill_gaps"] = gaps

    state["radar_data"] = [
        {"skill": g["skill"], "level": 3 if g["status"] == "已具备" else 1.5 if g["status"] == "部分具备" else 0}
        for g in gaps
    ]
    return state

async def step_learning_path(state: ProfileState) -> ProfileState:
    missing = [g for g in state["skill_gaps"] if g["status"] != "已具备"]
    prompt = f"""
目标岗位：{state['position_category']}
需要补上的技能（按优先级排序）：
{[g['skill'] for g in missing]}

请为每项技能生成：
1. 学习主题
2. 学到什么程度能达到面试要求
3. 推荐学习资源（3个以内）
4. 预估学习时长（小时）

以 JSON 返回：
[{{"topic": "", "target_level": "", "resources": ["", ""], "estimated_hours": 0}}]
"""
    data = await call_llm_structured(prompt)
    state["learning_path"] = data
    return state

def create_profile_agent():
    graph = StateGraph(ProfileState)
    graph.add_node("market_scan", step_market_scan)
    graph.add_node("gap_analysis", step_gap_analysis)
    graph.add_node("learning_path", step_learning_path)
    graph.set_entry_point("market_scan")
    graph.add_edge("market_scan", "gap_analysis")
    graph.add_edge("gap_analysis", "learning_path")
    graph.add_edge("learning_path", END)
    return graph.compile()

profile_agent = create_profile_agent()

async def run_profile_agent(
    position_category: str,
    user_skills: list,
    education: str,
    experience_years: int,
    db: AsyncSession,
):
    initial_state: ProfileState = {
        "position_category": position_category,
        "user_skills": user_skills,
        "education": education or "",
        "experience_years": experience_years,
        "common_skills": [],
        "skill_gaps": [],
        "learning_path": [],
        "radar_data": [],
    }
    result = await profile_agent.ainvoke(initial_state, {"db": db})
    return {
        "radar_data": result["radar_data"],
        "skill_gaps": result["skill_gaps"],
        "learning_path": result["learning_path"],
    }
```

- [ ] **Step 2: Create capability profile API**

```python
# backend/app/api/skills.py
from fastapi import APIRouter, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.database import get_db
from app.models.user import User
from app.schemas.jd import CapabilityProfileRequest, CapabilityProfileResponse
from app.middleware.auth import get_current_user
from app.services.agents.profile_agent import run_profile_agent

router = APIRouter(prefix="/api/skills", tags=["skills"])

@router.post("/profile", response_model=CapabilityProfileResponse)
async def generate_profile(
    body: CapabilityProfileRequest,
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    return await run_profile_agent(
        position_category=body.position_category,
        user_skills=body.user_skills,
        education=body.education or "",
        experience_years=body.experience_years,
        db=db,
    )
```

- [ ] **Step 3: Register router and verify**

Edit `backend/app/main.py`:
```python
from app.api.skills import router as skills_router
app.include_router(skills_router)
```

- [ ] **Step 4: Commit**

```bash
git add backend/app/services/agents/profile_agent.py backend/app/api/skills.py
git commit -m "feat: add capability profile agent with LangGraph and API"
```

---

## Phase 8: Resume Diagnostic Agent (Agent 2)

### Task 8.1: Resume Model and API

**Files:**
- Create: `backend/app/models/resume.py`
- Create: `backend/app/api/resumes.py`

- [ ] **Step 1: Create resume model**

```python
# backend/app/models/resume.py
from sqlalchemy import Column, Integer, String, DateTime, Text, JSON, ForeignKey, Float, func
from app.database import Base

class Resume(Base):
    __tablename__ = "resumes"

    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    original_content = Column(Text, nullable=False)
    jd_id = Column(Integer, ForeignKey("jd_raw.id"), nullable=True)
    jd_content = Column(Text, nullable=True)
    tailored_content = Column(Text)
    match_score = Column(Float)
    suggestions = Column(JSON, default=list)
    version = Column(Integer, default=1)
    created_at = Column(DateTime, server_default=func.now())
```

- [ ] **Step 2: Update models __init__ and run migration**

```python
# backend/app/models/__init__.py (add)
from app.models.resume import Resume
```

Run: `cd backend && alembic revision --autogenerate -m "add resumes table"`
Run: `cd backend && PYTHONPATH=. alembic upgrade head`

- [ ] **Step 3: Create resume schemas**

```python
# backend/app/schemas/resume.py
from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

class ResumeUpload(BaseModel):
    content: str  # extracted text from resume
    jd_id: Optional[int] = None
    jd_content: Optional[str] = None  # user-pasted JD if no jd_id

class MatchSuggestion(BaseModel):
    section: str  # which part of resume
    current: str
    suggested: str
    reason: str

class ResumeAnalysisResponse(BaseModel):
    match_score: float
    hard_skill_score: float
    soft_skill_score: float
    business_match_score: float
    suggestions: List[dict]
    hard_gaps: List[str]
    tailored_content: str

class ResumeListItem(BaseModel):
    id: int
    jd_id: Optional[int]
    match_score: Optional[float]
    version: int
    created_at: datetime

    class Config:
        from_attributes = True
```

- [ ] **Step 4: Commit**

```bash
git add backend/app/models/resume.py backend/app/schemas/resume.py backend/alembic/
git commit -m "feat: add Resume model and schemas"
```

---

### Task 8.2: Resume Diagnostic Agent

**Files:**
- Create: `backend/app/services/agents/resume_agent.py`

- [ ] **Step 1: Create resume diagnostic agent**

```python
# backend/app/services/agents/resume_agent.py
from typing import TypedDict, List
from langgraph.graph import StateGraph, END
from app.services.llm import call_llm_structured

class ResumeState(TypedDict):
    resume_text: str
    jd_text: str
    resume_skills: List[str]
    resume_projects: List[str]
    jd_requirements: List[str]
    jd_keywords: List[str]
    match_score: float
    hard_skill_score: float
    soft_skill_score: float
    business_match_score: float
    suggestions: List[dict]
    hard_gaps: List[str]
    tailored_content: str

async def step_parse(state: ResumeState) -> ResumeState:
    prompt = f"""
简历内容：
{state["resume_text"][:3000]}

提取：
1. skills: 技能标签列表
2. projects: 项目经历摘要列表，每个项目含 {"name": "", "description": ""}

JD 内容：
{state["jd_text"][:3000]}

提取：
3. requirements: 硬性要求列表
4. keywords: 高频关键词列表
"""
    data = await call_llm_structured(prompt)
    state["resume_skills"] = data.get("skills", [])
    state["resume_projects"] = data.get("projects", [])
    state["jd_requirements"] = data.get("requirements", [])
    state["jd_keywords"] = data.get("keywords", [])
    return state

async def step_score(state: ResumeState) -> ResumeState:
    resume_lower = " ".join(state["resume_skills"]).lower()
    jd_lower = " ".join(state["jd_requirements"]).lower()

    matched = sum(1 for req in state["jd_requirements"] if req.lower() in resume_lower)
    total = len(state["jd_requirements"]) or 1
    state["hard_skill_score"] = round(matched / total * 100)
    state["soft_skill_score"] = 70  # placeholder for LLM-based soft skill analysis
    state["business_match_score"] = 65
    state["match_score"] = round(
        state["hard_skill_score"] * 0.5 + state["soft_skill_score"] * 0.25 + state["business_match_score"] * 0.25
    )
    state["hard_gaps"] = [
        req for req in state["jd_requirements"]
        if req.lower() not in resume_lower
    ]
    return state

async def step_tailor(state: ResumeState) -> ResumeState:
    prompt = f"""
原始简历：{state["resume_text"][:2000]}
目标 JD 关键词：{state["jd_keywords"]}
硬伤（简历未提及的 JD 要求）：{state["hard_gaps"]}

请：
1. 为每段项目经历给出改写建议（关键词嵌入、动词强化）：[{{"section": "", "current": "", "suggested": "", "reason": ""}}]
2. 生成一份针对该 JD 优化后的简历全文
"""
    data = await call_llm_structured(prompt)
    state["suggestions"] = data.get("suggestions", data) if isinstance(data, list) else data.get("suggestions", [])
    state["tailored_content"] = data if isinstance(data, str) else data.get("tailored_content", "")
    if isinstance(state["tailored_content"], dict):
        state["tailored_content"] = state["resume_text"]
    return state

def create_resume_agent():
    graph = StateGraph(ResumeState)
    graph.add_node("parse", step_parse)
    graph.add_node("score", step_score)
    graph.add_node("tailor", step_tailor)
    graph.set_entry_point("parse")
    graph.add_edge("parse", "score")
    graph.add_edge("score", "tailor")
    graph.add_edge("tailor", END)
    return graph.compile()

resume_agent = create_resume_agent()

async def run_resume_agent(resume_text: str, jd_text: str):
    initial: ResumeState = {
        "resume_text": resume_text,
        "jd_text": jd_text,
        "resume_skills": [], "resume_projects": [],
        "jd_requirements": [], "jd_keywords": [],
        "match_score": 0, "hard_skill_score": 0,
        "soft_skill_score": 0, "business_match_score": 0,
        "suggestions": [], "hard_gaps": [],
        "tailored_content": "",
    }
    result = await resume_agent.ainvoke(initial)
    return {
        "match_score": result["match_score"],
        "hard_skill_score": result["hard_skill_score"],
        "soft_skill_score": result["soft_skill_score"],
        "business_match_score": result["business_match_score"],
        "suggestions": result["suggestions"],
        "hard_gaps": result["hard_gaps"],
        "tailored_content": result["tailored_content"],
    }
```

- [ ] **Step 2: Create resume API endpoints**

```python
# backend/app/api/resumes.py
from typing import List
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.database import get_db
from app.models.user import User
from app.models.resume import Resume
from app.models.jd import JDRaw
from app.schemas.resume import ResumeUpload, ResumeAnalysisResponse, ResumeListItem
from app.middleware.auth import get_current_user
from app.services.agents.resume_agent import run_resume_agent

router = APIRouter(prefix="/api/resumes", tags=["resumes"])

@router.post("/analyze", response_model=ResumeAnalysisResponse)
async def analyze_resume(
    body: ResumeUpload,
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    jd_text = body.jd_content or ""
    if body.jd_id and not jd_text:
        result = await db.execute(select(JDRaw).where(JDRaw.id == body.jd_id))
        jd = result.scalar_one_or_none()
        if jd:
            jd_text = jd.raw_text

    if not jd_text:
        raise HTTPException(status_code=400, detail="Please provide a JD (by ID or paste)")

    analysis = await run_resume_agent(body.content, jd_text)

    resume = Resume(
        user_id=user.id,
        original_content=body.content,
        jd_id=body.jd_id,
        jd_content=jd_text,
        tailored_content=analysis["tailored_content"],
        match_score=analysis["match_score"],
        suggestions=analysis["suggestions"],
    )
    db.add(resume)
    await db.commit()

    return analysis

@router.get("/", response_model=List[ResumeListItem])
async def list_resumes(
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    result = await db.execute(
        select(Resume).where(Resume.user_id == user.id).order_by(Resume.created_at.desc())
    )
    return result.scalars().all()

@router.get("/{resume_id}", response_model=ResumeAnalysisResponse)
async def get_resume(
    resume_id: int,
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    result = await db.execute(
        select(Resume).where(Resume.id == resume_id, Resume.user_id == user.id)
    )
    resume = result.scalar_one_or_none()
    if not resume:
        raise HTTPException(status_code=404)
    return {
        "match_score": resume.match_score or 0,
        "hard_skill_score": 0,
        "soft_skill_score": 0,
        "business_match_score": 0,
        "suggestions": resume.suggestions or [],
        "hard_gaps": [],
        "tailored_content": resume.tailored_content or "",
    }
```

- [ ] **Step 3: Register router**

Edit `backend/app/main.py`:
```python
from app.api.resumes import router as resumes_router
app.include_router(resumes_router)
```

- [ ] **Step 4: Commit**

```bash
git add backend/app/services/agents/resume_agent.py backend/app/api/resumes.py
git commit -m "feat: add resume diagnostic agent with scoring and tailoring"
```

---

## Phase 9: Interview Experiences Module

### Task 9.1: Interview Experience Model and Search

**Files:**
- Create: `backend/app/models/interview.py`
- Create: `backend/app/api/interviews.py`

- [ ] **Step 1: Create interview experience model**

```python
# backend/app/models/interview.py
from sqlalchemy import Column, Integer, String, DateTime, Text, JSON, ForeignKey, Float, func
from pgvector.sqlalchemy import Vector
from app.database import Base

class InterviewExperience(Base):
    __tablename__ = "interview_experiences"

    id = Column(Integer, primary_key=True, autoincrement=True)
    company = Column(String(200), nullable=False)
    position = Column(String(200), nullable=False)
    content_summary = Column(Text, nullable=False)
    raw_content = Column(Text)
    source_url = Column(String(500))
    source_platform = Column(String(50), nullable=False)
    tags = Column(JSON, default=list)
    embedding = Column(Vector(1536))
    published_at = Column(DateTime)
    crawled_at = Column(DateTime, server_default=func.now())

class MockInterview(Base):
    __tablename__ = "mock_interviews"

    id = Column(Integer, primary_key=True, autoincrement=True)
    user_id = Column(Integer, ForeignKey("users.id"), nullable=False)
    interview_type = Column(String(20), nullable=False)  # "general" or "company_specific"
    jd_id = Column(Integer, ForeignKey("jd_raw.id"), nullable=True)
    conversation = Column(JSON, default=list)
    score_report = Column(JSON)
    weak_points = Column(JSON, default=list)
    created_at = Column(DateTime, server_default=func.now())
```

- [ ] **Step 2: Update models __init__ and migrate**

```python
# backend/app/models/__init__.py (add)
from app.models.interview import InterviewExperience, MockInterview
```

Run: `cd backend && alembic revision --autogenerate -m "add interview_experiences and mock_interviews"`
Run: `cd backend && PYTHONPATH=. alembic upgrade head`

- [ ] **Step 3: Create interview search API**

```python
# backend/app/api/interviews.py
from typing import Optional, List
from fastapi import APIRouter, Depends, Query, HTTPException
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.database import get_db
from app.models.user import User
from app.models.interview import InterviewExperience, MockInterview
from app.middleware.auth import get_current_user
from app.services.llm import call_llm

router = APIRouter(prefix="/api/interviews", tags=["interviews"])

@router.get("/experiences")
async def search_experiences(
    company: Optional[str] = Query(None),
    position: Optional[str] = Query(None),
    platform: Optional[str] = Query(None),
    keyword: Optional[str] = Query(None),
    limit: int = Query(20, le=50),
    offset: int = Query(0),
    db: AsyncSession = Depends(get_db),
):
    q = select(InterviewExperience)
    if company:
        q = q.where(InterviewExperience.company.ilike(f"%{company}%"))
    if position:
        q = q.where(InterviewExperience.position.ilike(f"%{position}%"))
    if platform:
        q = q.where(InterviewExperience.source_platform == platform)
    if keyword:
        q = q.where(InterviewExperience.content_summary.ilike(f"%{keyword}%"))
    q = q.order_by(InterviewExperience.crawled_at.desc()).offset(offset).limit(limit)
    result = await db.execute(q)
    experiences = result.scalars().all()
    return [
        {
            "id": e.id,
            "company": e.company,
            "position": e.position,
            "content_summary": e.content_summary,
            "source_url": e.source_url,
            "source_platform": e.source_platform,
            "tags": e.tags,
            "published_at": e.published_at and e.published_at.isoformat(),
            "crawled_at": e.crawled_at.isoformat() if e.crawled_at else None,
        }
        for e in experiences
    ]

@router.get("/experiences/{exp_id}")
async def get_experience_detail(exp_id: int, db: AsyncSession = Depends(get_db)):
    result = await db.execute(select(InterviewExperience).where(InterviewExperience.id == exp_id))
    exp = result.scalar_one_or_none()
    if not exp:
        raise HTTPException(status_code=404)
    summary = exp.content_summary
    if not summary and exp.raw_content:
        prompt = f"请用200字以内概括以下面经的核心要点：\n{exp.raw_content[:3000]}"
        summary = await call_llm(prompt)
        exp.content_summary = summary
        await db.commit()
    return {
        "id": exp.id,
        "company": exp.company,
        "position": exp.position,
        "content_summary": summary,
        "raw_content": exp.raw_content,
        "source_url": exp.source_url,
        "source_platform": exp.source_platform,
        "tags": exp.tags,
        "published_at": exp.published_at and exp.published_at.isoformat(),
    }
```

- [ ] **Step 4: Register router**

Edit `backend/app/main.py`:
```python
from app.api.interviews import router as interviews_router
app.include_router(interviews_router)
```

- [ ] **Step 5: Commit**

```bash
git add backend/app/models/interview.py backend/app/api/interviews.py backend/alembic/
git commit -m "feat: add interview experience model and search API"
```

---

## Phase 10: Mock Interview Agent (Agent 3)

### Task 10.1: Interview Agent with LangGraph

**Files:**
- Create: `backend/app/services/agents/interview_agent.py`

- [ ] **Step 1: Create interview agent**

```python
# backend/app/services/agents/interview_agent.py
from typing import TypedDict, List
from langgraph.graph import StateGraph, END
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.models.interview import InterviewExperience
from app.models.jd import JDRaw
from app.services.llm import call_llm, call_llm_structured

class InterviewState(TypedDict):
    interview_type: str
    position: str
    company: str
    jd_id: int
    conversation: List[dict]
    current_stage: str
    stages: List[str]
    system_prompt: str
    score_report: dict
    weak_points: List[str]

STAGES = ["self_intro", "technical_basics", "project_deep_dive", "scenario_question", "reversed_question"]

async def step_init_role(state: InterviewState, db: AsyncSession) -> InterviewState:
    if state["interview_type"] == "company_specific" and state["company"]:
        result = await db.execute(
            select(InterviewExperience)
            .where(InterviewExperience.company.ilike(f"%{state['company']}%"))
            .limit(20)
        )
        experiences = result.scalars().all()
        exp_summaries = "\n".join([e.content_summary[:500] for e in experiences])
        system_prompt = f"""你是一位{state['company']}的面试官，正在面试{state['position']}岗位。
以下是该公司近期面经：
{exp_summaries}

面试流程：自我介绍 → 基础知识 → 项目深挖 → 场景题 → 反问。
请根据该公司真实面试风格出题，难度循序渐进。
每次只问一个问题。根据候选人回答质量决定追问还是进入下一阶段。
"""
    else:
        system_prompt = f"""你是一位资深面试官，正在面试{state['position']}岗位。
面试流程：自我介绍 → 基础知识 → 项目深挖 → 场景题 → 反问。
每次只问一个问题。根据候选人回答质量决定追问还是进入下一阶段。
"""
    state["system_prompt"] = system_prompt
    state["stages"] = STAGES.copy()
    state["current_stage"] = "self_intro"
    state["conversation"] = [{"role": "interviewer", "content": f"你好，欢迎参加{state['position']}的面试。请先简单自我介绍一下。"}]
    return state

async def step_next_question(state: InterviewState) -> InterviewState:
    last_answer = ""
    for msg in reversed(state["conversation"]):
        if msg["role"] == "candidate":
            last_answer = msg["content"]
            break

    if not last_answer:
        state["conversation"].append({"role": "interviewer", "content": "请回答我的问题。"})
        return state

    # Decide whether to follow up or move to next stage
    prompt = f"""当前面试阶段：{state['current_stage']}
候选人回答：{last_answer}

判断：
1. 如果回答太简略或不够深入，给出一个追问
2. 如果回答充分，说"NEXT_STAGE"进入下一阶段
注意：如果是反问阶段(scenario_question)，回答完就说"END"结束面试
"""
    decision = await call_llm(prompt, state["system_prompt"])
    decision = decision.strip()

    if "NEXT_STAGE" in decision or "END" in decision:
        current_idx = state["stages"].index(state["current_stage"])
        if current_idx < len(state["stages"]) - 1:
            state["current_stage"] = state["stages"][current_idx + 1]
            stage_prompts = {
                "technical_basics": "接下来考察基础知识，请听第一题。",
                "project_deep_dive": "接下来请介绍一下你最熟悉的一个项目。",
                "scenario_question": "接下来是一道场景题。",
                "reversed_question": "面试接近尾声，你有什么想问我的吗？",
            }
            transition = stage_prompts.get(state["current_stage"], "我们继续。")
            state["conversation"].append({"role": "interviewer", "content": transition})
        else:
            state["conversation"].append({"role": "interviewer", "content": "面试结束，谢谢你的时间。"})
    else:
        state["conversation"].append({"role": "interviewer", "content": decision})
    return state

async def step_score(state: InterviewState) -> InterviewState:
    dialogue = "\n".join([f"{m['role']}: {m['content']}" for m in state["conversation"]])
    prompt = f"""面试对话：
{dialogue}

请从以下维度打分(1-10)，并指出2-3个具体改进点：
1. 技术深度
2. 表达清晰度
3. 项目理解
4. 临场反应

返回 JSON: {{"technical_depth": 7, "clarity": 7, "project_understanding": 7, "response": 7, "improvements": ["改进点1", "改进点2"], "weak_points": ["薄弱项1"]}}
"""
    data = await call_llm_structured(prompt)
    state["score_report"] = data
    state["weak_points"] = data.get("weak_points", [])
    return state

def create_interview_agent():
    graph = StateGraph(InterviewState)
    graph.add_node("init_role", step_init_role)
    graph.add_node("next_question", step_next_question)
    graph.add_node("score", step_score)
    graph.set_entry_point("init_role")
    graph.add_edge("init_role", "next_question")

    def should_continue(state: InterviewState):
        last_msg = state["conversation"][-1]["content"]
        if "面试结束" in last_msg:
            return "score"
        # In practice, the caller loops: agent.invoke → add user answer → agent.invoke
        return END

    graph.add_conditional_edges("next_question", should_continue, {"score": "score", END: END})
    graph.add_edge("score", END)
    return graph.compile()

interview_agent = create_interview_agent()

async def start_interview(
    interview_type: str,
    position: str,
    company: str,
    jd_id: int,
    db: AsyncSession,
):
    initial: InterviewState = {
        "interview_type": interview_type,
        "position": position,
        "company": company,
        "jd_id": jd_id,
        "conversation": [],
        "current_stage": "",
        "stages": STAGES.copy(),
        "system_prompt": "",
        "score_report": {},
        "weak_points": [],
    }
    return await interview_agent.ainvoke(initial, {"db": db})

async def continue_interview(state: InterviewState, user_answer: str):
    state["conversation"].append({"role": "candidate", "content": user_answer})
    return await interview_agent.ainvoke(state)
```

- [ ] **Step 2: Add mock interview API endpoints**

```python
# backend/app/api/interviews.py (add to existing file)

from app.services.agents.interview_agent import start_interview, continue_interview

@router.post("/mock/start")
async def start_mock_interview(
    interview_type: str = "general",
    position: str = "后端开发工程师",
    company: str = "",
    jd_id: int = None,
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    result = await start_interview(
        interview_type=interview_type,
        position=position,
        company=company,
        jd_id=jd_id,
        db=db,
    )

    interview = MockInterview(
        user_id=user.id,
        interview_type=interview_type,
        jd_id=jd_id,
        conversation=result["conversation"],
    )
    db.add(interview)
    await db.commit()
    await db.refresh(interview)

    return {
        "interview_id": interview.id,
        "conversation": result["conversation"],
        "current_stage": result["current_stage"],
    }

@router.post("/mock/{interview_id}/respond")
async def respond_to_interview(
    interview_id: int,
    answer: dict,
    user: User = Depends(get_current_user),
    db: AsyncSession = Depends(get_db),
):
    result = await db.execute(
        select(MockInterview).where(
            MockInterview.id == interview_id,
            MockInterview.user_id == user.id,
        )
    )
    interview = result.scalar_one_or_none()
    if not interview:
        raise HTTPException(status_code=404)

    from app.services.agents.interview_agent import InterviewState
    state: InterviewState = {
        "interview_type": interview.interview_type,
        "position": "",
        "company": "",
        "jd_id": interview.jd_id,
        "conversation": interview.conversation or [],
        "current_stage": "",
        "stages": [],
        "system_prompt": "",
        "score_report": {},
        "weak_points": [],
    }

    result = await continue_interview(state, answer.get("content", ""))

    last_msg = result["conversation"][-1]["content"] if result["conversation"] else ""
    is_finished = "面试结束" in last_msg

    if is_finished:
        interview.conversation = result["conversation"]
        interview.score_report = result.get("score_report", {})
        interview.weak_points = result.get("weak_points", [])
        await db.commit()
        return {
            "finished": True,
            "conversation": result["conversation"],
            "score_report": result.get("score_report", {}),
            "weak_points": result.get("weak_points", []),
        }

    interview.conversation = result["conversation"]
    await db.commit()
    return {
        "finished": False,
        "conversation": result["conversation"],
        "current_stage": result.get("current_stage", ""),
    }
```

- [ ] **Step 3: Register updated routes**

The interview router is already registered. No additional main.py changes.

- [ ] **Step 4: Commit**

```bash
git add backend/app/services/agents/interview_agent.py backend/app/api/interviews.py
git commit -m "feat: add mock interview agent with multi-stage conversation flow"
```

---

## Phase 11: Frontend Pages

### Task 11.1: Dashboard Page

**Files:**
- Create: `frontend/src/pages/Dashboard.tsx`

- [ ] **Step 1: Create Dashboard**

```tsx
// frontend/src/pages/Dashboard.tsx
import { useEffect, useState } from "react";
import { Link } from "react-router-dom";
import api from "../api/client";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";

export default function Dashboard() {
  const [stats, setStats] = useState({ total_jd_count: 0, recent_jd_count: 0, recent_crawl_count: 0 });
  const [crawlLogs, setCrawlLogs] = useState([]);

  useEffect(() => {
    api.get("/dashboard/stats").then((r) => setStats(r.data));
    api.get("/dashboard/crawl-logs").then((r) => setCrawlLogs(r.data));
  }, []);

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">仪表盘</h1>

      <div className="grid grid-cols-3 gap-4">
        <Card>
          <CardHeader className="pb-2"><CardTitle className="text-sm text-gray-500">岗位总数</CardTitle></CardHeader>
          <CardContent><p className="text-3xl font-bold">{stats.total_jd_count}</p></CardContent>
        </Card>
        <Card>
          <CardHeader className="pb-2"><CardTitle className="text-sm text-gray-500">今日新增 JD</CardTitle></CardHeader>
          <CardContent><p className="text-3xl font-bold text-green-600">+{stats.recent_jd_count}</p></CardContent>
        </Card>
        <Card>
          <CardHeader className="pb-2"><CardTitle className="text-sm text-gray-500">今日爬取次数</CardTitle></CardHeader>
          <CardContent><p className="text-3xl font-bold">{stats.recent_crawl_count}</p></CardContent>
        </Card>
      </div>

      <div className="grid grid-cols-2 gap-6">
        <Card>
          <CardHeader><CardTitle>最近爬取动态</CardTitle></CardHeader>
          <CardContent>
            {crawlLogs.length === 0 && <p className="text-gray-400">暂无爬取记录</p>}
            <div className="space-y-2">
              {(crawlLogs as any[]).slice(0, 10).map((log: any) => (
                <div key={log.id} className="flex items-center justify-between py-1 border-b last:border-0 text-sm">
                  <span>
                    <Badge variant={log.status === "success" ? "default" : "destructive"} className="mr-2">
                      {log.platform}
                    </Badge>
                    关键词: {log.keyword}
                  </span>
                  <span className="text-gray-400">
                    新增 {log.new_count} 条 · {new Date(log.crawled_at).toLocaleTimeString()}
                  </span>
                </div>
              ))}
            </div>
          </CardContent>
        </Card>

        <Card>
          <CardHeader><CardTitle>下一步建议</CardTitle></CardHeader>
          <CardContent className="space-y-3">
            <Link to="/positions/profile" className="block">
              <Button variant="outline" className="w-full justify-start">生成我的能力画像和学习路线</Button>
            </Link>
            <Link to="/resumes" className="block">
              <Button variant="outline" className="w-full justify-start">上传简历，针对目标 JD 做匹配分析</Button>
            </Link>
            <Link to="/interviews/mock" className="block">
              <Button variant="outline" className="w-full justify-start">开始一次模拟面试</Button>
            </Link>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

- [ ] **Step 2: Commit**

```bash
git add frontend/src/pages/Dashboard.tsx
git commit -m "feat: add dashboard page with stats, crawl logs, and next-step suggestions"
```

---

### Task 11.2: Market Overview and Capability Profile Pages

**Files:**
- Create: `frontend/src/pages/MarketOverview.tsx`
- Create: `frontend/src/pages/CapabilityProfile.tsx`

- [ ] **Step 1: Create Market Overview page**

```tsx
// frontend/src/pages/MarketOverview.tsx
import { useEffect, useState } from "react";
import api from "../api/client";
import { Card, CardContent } from "@/components/ui/card";
import { Badge } from "@/components/ui/badge";
import { Input } from "@/components/ui/input";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";

export default function MarketOverview() {
  const [jds, setJds] = useState([]);
  const [categories, setCategories] = useState([]);
  const [selectedCategory, setSelectedCategory] = useState("");
  const [keyword, setKeyword] = useState("");

  useEffect(() => {
    api.get("/positions/categories").then((r) => setCategories(r.data));
  }, []);

  const search = () => {
    const params = new URLSearchParams();
    if (selectedCategory) params.append("category", selectedCategory);
    if (keyword) params.append("keyword", keyword);
    api.get(`/positions/jd?${params}`).then((r) => setJds(r.data));
  };

  useEffect(() => { search(); }, [selectedCategory]);

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">市场全景</h1>

      <div className="flex gap-4">
        <Select onValueChange={setSelectedCategory} value={selectedCategory}>
          <SelectTrigger className="w-48">
            <SelectValue placeholder="选择岗位类型" />
          </SelectTrigger>
          <SelectContent>
            {(categories as any[]).map((c: any) => (
              <SelectItem key={c.category} value={c.category}>{c.category} ({c.count})</SelectItem>
            ))}
          </SelectContent>
        </Select>
        <Input placeholder="搜索关键词" value={keyword} onChange={(e) => setKeyword(e.target.value)}
          onKeyDown={(e) => e.key === "Enter" && search()} className="flex-1" />
      </div>

      <div className="grid gap-4">
        {(jds as any[]).map((jd: any) => (
          <Card key={jd.id}>
            <CardContent className="p-4">
              <div className="flex items-start justify-between">
                <div>
                  <h3 className="font-semibold">{jd.title}</h3>
                  <p className="text-sm text-gray-500">{jd.company} · {jd.city} · {jd.salary_range}</p>
                  <p className="mt-2 text-sm text-gray-700 line-clamp-3">{jd.raw_text}</p>
                </div>
                <div className="flex flex-col gap-1 items-end shrink-0">
                  <Badge>{jd.source_platform}</Badge>
                  <span className="text-xs text-gray-400">{new Date(jd.crawled_at).toLocaleDateString()}</span>
                </div>
              </div>
              <a href={jd.source_url} target="_blank" className="text-xs text-blue-500 mt-2 inline-block">查看原文</a>
            </CardContent>
          </Card>
        ))}
      </div>
    </div>
  );
}
```

- [ ] **Step 2: Create Capability Profile page**

```tsx
// frontend/src/pages/CapabilityProfile.tsx
import { useState } from "react";
import api from "../api/client";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";

export default function CapabilityProfile() {
  const [position, setPosition] = useState("");
  const [skillInput, setSkillInput] = useState("");
  const [skills, setSkills] = useState<string[]>([]);
  const [education, setEducation] = useState("");
  const [years, setYears] = useState(0);
  const [result, setResult] = useState<any>(null);
  const [loading, setLoading] = useState(false);

  const addSkill = () => {
    if (skillInput.trim()) {
      setSkills([...skills, skillInput.trim()]);
      setSkillInput("");
    }
  };

  const analyze = async () => {
    setLoading(true);
    const { data } = await api.post("/skills/profile", {
      position_category: position,
      user_skills: skills,
      education,
      experience_years: years,
    });
    setResult(data);
    setLoading(false);
  };

  return (
    <div className="space-y-6 max-w-4xl">
      <h1 className="text-2xl font-bold">能力画像 & 学习路线</h1>

      <Card>
        <CardHeader><CardTitle>我的背景</CardTitle></CardHeader>
        <CardContent className="space-y-4">
          <div className="flex gap-4">
            <Input placeholder="目标岗位 (如: 后端开发工程师)" value={position} onChange={(e) => setPosition(e.target.value)} />
            <Input placeholder="学历" value={education} onChange={(e) => setEducation(e.target.value)} />
            <Input type="number" placeholder="工作年限" value={years} onChange={(e) => setYears(Number(e.target.value))} />
          </div>
          <div className="flex gap-2">
            <Input placeholder="添加技能 (如: Python)" value={skillInput}
              onChange={(e) => setSkillInput(e.target.value)} onKeyDown={(e) => e.key === "Enter" && addSkill()} />
            <Button variant="outline" onClick={addSkill}>添加</Button>
          </div>
          <div className="flex gap-1 flex-wrap">
            {skills.map((s, i) => (
              <Badge key={i} variant="secondary" className="cursor-pointer" onClick={() => setSkills(skills.filter((_, j) => j !== i))}>
                {s} x
              </Badge>
            ))}
          </div>
          <Button onClick={analyze} disabled={loading || !position || skills.length === 0}>
            {loading ? "分析中..." : "生成能力画像"}
          </Button>
        </CardContent>
      </Card>

      {result && (
        <>
          <Card>
            <CardHeader><CardTitle>技能雷达</CardTitle></CardHeader>
            <CardContent>
              <div className="space-y-2">
                {(result.skill_gaps || []).map((gap: any) => (
                  <div key={gap.skill} className="flex items-center gap-3">
                    <span className="w-24 text-sm">{gap.skill}</span>
                    <div className="flex-1 h-4 bg-gray-100 rounded-full overflow-hidden">
                      <div className={`h-full rounded-full ${gap.status === "已具备" ? "bg-green-500" : gap.status === "部分具备" ? "bg-yellow-500" : "bg-red-400"}`}
                        style={{ width: gap.status === "已具备" ? "100%" : gap.status === "部分具备" ? "50%" : "15%" }} />
                    </div>
                    <Badge variant={gap.status === "已具备" ? "default" : gap.status === "部分具备" ? "secondary" : "destructive"}>
                      {gap.status}
                    </Badge>
                  </div>
                ))}
              </div>
            </CardContent>
          </Card>

          <Card>
            <CardHeader><CardTitle>学习路线</CardTitle></CardHeader>
            <CardContent>
              <div className="space-y-4">
                {(result.learning_path || []).map((item: any, i: number) => (
                  <div key={i} className="border-l-2 border-blue-500 pl-4 py-2">
                    <h4 className="font-medium">{item.topic}</h4>
                    <p className="text-sm text-gray-500">目标: {item.target_level}</p>
                    <p className="text-sm text-gray-500">
                      资源: {(item.resources || []).join(" / ")}
                    </p>
                    <Badge variant="outline" className="mt-1">预计 {item.estimated_hours}h</Badge>
                  </div>
                ))}
              </div>
            </CardContent>
          </Card>
        </>
      )}
    </div>
  );
}
```

- [ ] **Step 3: Commit**

```bash
git add frontend/src/pages/MarketOverview.tsx frontend/src/pages/CapabilityProfile.tsx
git commit -m "feat: add market overview and capability profile pages"
```

---

### Task 11.3: Resume Tailor and Interview Pages

**Files:**
- Create: `frontend/src/pages/ResumeTailor.tsx`
- Create: `frontend/src/pages/InterviewSearch.tsx`
- Create: `frontend/src/pages/MockInterview.tsx`
- Create: `frontend/src/pages/InterviewReport.tsx`

- [ ] **Step 1: Create Resume Tailor page**

```tsx
// frontend/src/pages/ResumeTailor.tsx
import { useState } from "react";
import api from "../api/client";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Badge } from "@/components/ui/badge";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";

export default function ResumeTailor() {
  const [resume, setResume] = useState("");
  const [jdId, setJdId] = useState("");
  const [jdPaste, setJdPaste] = useState("");
  const [result, setResult] = useState<any>(null);
  const [versions, setVersions] = useState<any[]>([]);
  const [loading, setLoading] = useState(false);

  const analyze = async () => {
    setLoading(true);
    const { data } = await api.post("/resumes/analyze", {
      content: resume,
      jd_id: jdId ? Number(jdId) : undefined,
      jd_content: jdPaste || undefined,
    });
    setResult(data);
    setLoading(false);
  };

  const loadVersions = async () => {
    const { data } = await api.get("/resumes/");
    setVersions(data);
  };

  return (
    <div className="space-y-6 max-w-4xl">
      <h1 className="text-2xl font-bold">简历包装</h1>

      <Card>
        <CardHeader><CardTitle>上传简历 & 选择 JD</CardTitle></CardHeader>
        <CardContent className="space-y-4">
          <Textarea placeholder="粘贴简历全文..." value={resume} onChange={(e) => setResume(e.target.value)} rows={8} />
          <div className="flex gap-4">
            <input type="number" placeholder="JD ID (从岗位洞察中选择)" value={jdId}
              onChange={(e) => setJdId(e.target.value)} className="flex-1 px-3 py-2 border rounded" />
            <span className="text-gray-400 py-2">或</span>
          </div>
          <Textarea placeholder="或直接粘贴 JD 文本..." value={jdPaste} onChange={(e) => setJdPaste(e.target.value)} rows={4} />
          <Button onClick={analyze} disabled={loading || !resume || (!jdId && !jdPaste)}>
            {loading ? "分析中..." : "分析匹配度"}
          </Button>
        </CardContent>
      </Card>

      {result && (
        <>
          <Card>
            <CardHeader><CardTitle>匹配度报告</CardTitle></CardHeader>
            <CardContent>
              <div className="flex items-center gap-6 mb-4">
                <div className="text-center">
                  <div className="text-4xl font-bold text-blue-600">{result.match_score}%</div>
                  <p className="text-sm text-gray-500">综合匹配</p>
                </div>
                <div className="flex-1 space-y-1">
                  <div className="flex justify-between text-sm"><span>硬技能</span><span>{result.hard_skill_score}%</span></div>
                  <div className="h-2 bg-gray-100 rounded-full"><div className="h-2 bg-blue-500 rounded-full" style={{ width: `${result.hard_skill_score}%` }} /></div>
                  <div className="flex justify-between text-sm"><span>软素质</span><span>{result.soft_skill_score}%</span></div>
                  <div className="h-2 bg-gray-100 rounded-full"><div className="h-2 bg-green-500 rounded-full" style={{ width: `${result.soft_skill_score}%` }} /></div>
                  <div className="flex justify-between text-sm"><span>业务匹配</span><span>{result.business_match_score}%</span></div>
                  <div className="h-2 bg-gray-100 rounded-full"><div className="h-2 bg-yellow-500 rounded-full" style={{ width: `${result.business_match_score}%` }} /></div>
                </div>
              </div>

              {result.hard_gaps?.length > 0 && (
                <div className="mt-4">
                  <h4 className="font-medium text-red-600 mb-2">硬伤提醒</h4>
                  <div className="flex gap-1 flex-wrap">
                    {result.hard_gaps.map((gap: string, i: number) => (
                      <Badge key={i} variant="destructive">{gap}</Badge>
                    ))}
                  </div>
                </div>
              )}
            </CardContent>
          </Card>

          <Card>
            <CardHeader><CardTitle>改写建议</CardTitle></CardHeader>
            <CardContent>
              {(result.suggestions || []).map((s: any, i: number) => (
                <div key={i} className="mb-4 p-3 bg-gray-50 rounded">
                  <p className="font-medium">{s.section}</p>
                  <p className="text-sm text-gray-500 line-through">原: {s.current}</p>
                  <p className="text-sm text-green-700">建议: {s.suggested}</p>
                  <p className="text-xs text-gray-400 mt-1">原因: {s.reason}</p>
                </div>
              ))}
            </CardContent>
          </Card>

          <Card>
            <CardHeader><CardTitle>优化后简历</CardTitle></CardHeader>
            <CardContent>
              <Textarea value={result.tailored_content} readOnly rows={12} className="font-mono text-sm" />
            </CardContent>
          </Card>
        </>
      )}

      <Card>
        <CardHeader>
          <CardTitle className="flex justify-between items-center">
            历史版本
            <Button variant="outline" size="sm" onClick={loadVersions}>刷新</Button>
          </CardTitle>
        </CardHeader>
        <CardContent>
          {versions.map((v: any) => (
            <div key={v.id} className="flex justify-between py-2 border-b last:border-0 text-sm">
              <span>版本 {v.version}</span>
              <span>匹配度: {v.match_score}%</span>
              <span className="text-gray-400">{new Date(v.created_at).toLocaleDateString()}</span>
            </div>
          ))}
        </CardContent>
      </Card>
    </div>
  );
}
```

- [ ] **Step 2: Create Interview Search page**

```tsx
// frontend/src/pages/InterviewSearch.tsx
import { useState } from "react";
import api from "../api/client";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Badge } from "@/components/ui/badge";
import { Dialog, DialogContent, DialogHeader, DialogTitle, DialogTrigger } from "@/components/ui/dialog";

export default function InterviewSearch() {
  const [results, setResults] = useState<any[]>([]);
  const [company, setCompany] = useState("");
  const [position, setPosition] = useState("");
  const [detail, setDetail] = useState<any>(null);

  const search = async () => {
    const params = new URLSearchParams();
    if (company) params.append("company", company);
    if (position) params.append("position", position);
    const { data } = await api.get(`/interviews/experiences?${params}`);
    setResults(data);
  };

  const viewDetail = async (id: number) => {
    const { data } = await api.get(`/interviews/experiences/${id}`);
    setDetail(data);
  };

  return (
    <div className="space-y-6">
      <h1 className="text-2xl font-bold">面经知识库</h1>

      <div className="flex gap-4">
        <Input placeholder="公司名" value={company} onChange={(e) => setCompany(e.target.value)} />
        <Input placeholder="岗位" value={position} onChange={(e) => setPosition(e.target.value)} />
        <Button onClick={search}>搜索</Button>
      </div>

      <div className="grid gap-4">
        {results.map((r) => (
          <Card key={r.id}>
            <CardContent className="p-4">
              <div className="flex justify-between items-start">
                <div>
                  <h3 className="font-semibold">{r.company} - {r.position}</h3>
                  <p className="text-sm text-gray-600 mt-1">{r.content_summary?.slice(0, 200)}</p>
                  <div className="flex gap-1 mt-2">
                    {(r.tags || []).slice(0, 5).map((t: string, i: number) => (
                      <Badge key={i} variant="secondary" className="text-xs">{t}</Badge>
                    ))}
                  </div>
                </div>
                <div className="flex flex-col gap-2 items-end shrink-0">
                  <Badge>{r.source_platform}</Badge>
                  <span className="text-xs text-gray-400">{new Date(r.published_at || r.crawled_at).toLocaleDateString()}</span>
                  <a href={r.source_url} target="_blank" className="text-xs text-blue-500">原文链接</a>
                  <Dialog>
                    <DialogTrigger asChild>
                      <Button size="sm" variant="outline" onClick={() => viewDetail(r.id)}>查看详情</Button>
                    </DialogTrigger>
                    <DialogContent className="max-w-2xl max-h-[80vh] overflow-y-auto">
                      <DialogHeader><DialogTitle>{r.company} - {r.position} 面经</DialogTitle></DialogHeader>
                      {detail && <div className="text-sm whitespace-pre-wrap">{detail.raw_content || detail.content_summary}</div>}
                    </DialogContent>
                  </Dialog>
                </div>
              </div>
            </CardContent>
          </Card>
        ))}
      </div>
    </div>
  );
}
```

- [ ] **Step 3: Create Mock Interview page**

```tsx
// frontend/src/pages/MockInterview.tsx
import { useState, useRef, useEffect } from "react";
import { useNavigate } from "react-router-dom";
import api from "../api/client";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Avatar } from "@/components/ui/avatar";

export default function MockInterview() {
  const [type, setType] = useState("general");
  const [position, setPosition] = useState("");
  const [company, setCompany] = useState("");
  const [active, setActive] = useState(false);
  const [interviewId, setInterviewId] = useState<number | null>(null);
  const [messages, setMessages] = useState<any[]>([]);
  const [answer, setAnswer] = useState("");
  const [report, setReport] = useState<any>(null);
  const chatRef = useRef<HTMLDivElement>(null);

  const start = async () => {
    const { data } = await api.post("/interviews/mock/start", null, {
      params: { interview_type: type, position, company },
    });
    setInterviewId(data.interview_id);
    setMessages(data.conversation || []);
    setActive(true);
  };

  const respond = async () => {
    if (!answer.trim() || !interviewId) return;
    const { data } = await api.post(`/interviews/mock/${interviewId}/respond`, { content: answer });
    setMessages(data.conversation || []);
    setAnswer("");
    if (data.finished) {
      setActive(false);
      setReport(data.score_report);
    }
  };

  useEffect(() => {
    chatRef.current?.scrollTo(0, chatRef.current.scrollHeight);
  }, [messages]);

  if (report) {
    return (
      <div className="max-w-2xl mx-auto space-y-6">
        <h1 className="text-2xl font-bold">面试报告</h1>
        <Card>
          <CardHeader><CardTitle>评分</CardTitle></CardHeader>
          <CardContent>
            <div className="grid grid-cols-2 gap-4">
              {Object.entries(report).filter(([k]) => k !== "improvements" && k !== "weak_points").map(([k, v]) => (
                <div key={k} className="text-center p-4 bg-gray-50 rounded">
                  <div className="text-3xl font-bold text-blue-600">{v as number}/10</div>
                  <p className="text-sm text-gray-500">{k}</p>
                </div>
              ))}
            </div>
            <div className="mt-4">
              <h4 className="font-medium">改进建议</h4>
              <ul className="list-disc list-inside text-sm text-gray-600 mt-2">
                {(report.improvements || []).map((imp: string, i: number) => (
                  <li key={i}>{imp}</li>
                ))}
              </ul>
            </div>
          </CardContent>
        </Card>
        <Button onClick={() => { setReport(null); setMessages([]); setActive(false); }}>再来一次</Button>
      </div>
    );
  }

  if (!active) {
    return (
      <div className="max-w-md mx-auto space-y-6">
        <h1 className="text-2xl font-bold">模拟面试</h1>
        <Card>
          <CardHeader><CardTitle>面试设置</CardTitle></CardHeader>
          <CardContent className="space-y-4">
            <Select onValueChange={setType} value={type}>
              <SelectTrigger><SelectValue /></SelectTrigger>
              <SelectContent>
                <SelectItem value="general">通用面试</SelectItem>
                <SelectItem value="company_specific">具体公司面试</SelectItem>
              </SelectContent>
            </Select>
            <Input placeholder="目标岗位" value={position} onChange={(e) => setPosition(e.target.value)} />
            {type === "company_specific" && (
              <Input placeholder="公司名" value={company} onChange={(e) => setCompany(e.target.value)} />
            )}
            <Button onClick={start} disabled={!position} className="w-full">开始面试</Button>
          </CardContent>
        </Card>
      </div>
    );
  }

  return (
    <div className="max-w-2xl mx-auto space-y-4">
      <h1 className="text-2xl font-bold">模拟面试中</h1>
      <Card>
        <CardContent className="p-4">
          <div ref={chatRef} className="h-96 overflow-y-auto space-y-4 mb-4">
            {messages.map((m, i) => (
              <div key={i} className={`flex ${m.role === "candidate" ? "justify-end" : "justify-start"}`}>
                <div className={`max-w-[80%] p-3 rounded-lg ${m.role === "interviewer" ? "bg-gray-100" : "bg-blue-500 text-white"}`}>
                  <p className="text-xs mb-1 opacity-70">{m.role === "interviewer" ? "面试官" : "我"}</p>
                  <p className="whitespace-pre-wrap text-sm">{m.content}</p>
                </div>
              </div>
            ))}
          </div>
          <div className="flex gap-2">
            <Textarea value={answer} onChange={(e) => setAnswer(e.target.value)}
              onKeyDown={(e) => { if (e.key === "Enter" && !e.shiftKey) { e.preventDefault(); respond(); } }}
              placeholder="输入你的回答... (Enter 发送, Shift+Enter 换行)" rows={2} />
            <Button onClick={respond} disabled={!answer.trim()}>发送</Button>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
```

- [ ] **Step 4: Update App.tsx with all routes**

```tsx
// frontend/src/App.tsx (update Routes)
import MarketOverview from "./pages/MarketOverview";
import CapabilityProfile from "./pages/CapabilityProfile";
import ResumeTailor from "./pages/ResumeTailor";
import InterviewSearch from "./pages/InterviewSearch";
import MockInterview from "./pages/MockInterview";

// Inside Routes, add:
<Route path="/positions/market" element={<MarketOverview />} />
<Route path="/positions/profile" element={<CapabilityProfile />} />
<Route path="/resumes" element={<ResumeTailor />} />
<Route path="/interviews/experiences" element={<InterviewSearch />} />
<Route path="/interviews/mock" element={<MockInterview />} />
```

- [ ] **Step 5: Verify frontend compiles**

Run: `cd frontend && npm run build`
Expected: Build succeeds with no errors

- [ ] **Step 6: Commit**

```bash
git add frontend/src/pages/ frontend/src/App.tsx
git commit -m "feat: add resume tailor, interview search, and mock interview pages"
```

---

## Phase 12: Deployment Configuration

### Task 12.1: Docker Compose and Deployment Docs

**Files:**
- Create: `nginx.conf`
- Create: `frontend/Dockerfile`
- Modify: `docker-compose.yml`

- [ ] **Step 1: Create frontend Dockerfile**

```dockerfile
# frontend/Dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

- [ ] **Step 2: Create nginx config**

```nginx
# nginx.conf
server {
    listen 80;
    server_name _;

    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api {
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

- [ ] **Step 3: Update docker-compose.yml to include frontend**

Add to docker-compose.yml services:
```yaml
  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend
```

- [ ] **Step 4: Commit final deployment config**

```bash
git add nginx.conf frontend/Dockerfile docker-compose.yml
git commit -m "feat: add Docker Compose production deployment with nginx frontend"
```

---

## Summary: Task Completion Order

1. Task 1.1 → 1.2: Project scaffolding (backend + frontend)
2. Task 2.1 → 2.2 → 2.3: Auth system (models → API → profile)
3. Task 3.1: Frontend auth pages and routing
4. Task 4.1 → 4.2 → 4.3 → 4.4: Data pipeline (models → crawler base → Boss crawler → dashboard API)
5. Task 5.1: JD listing API
6. Task 6.1 → 6.2: LLM service + JD analyzer
7. Task 7.1: Capability profile Agent
8. Task 8.1 → 8.2: Resume model + diagnostic Agent
9. Task 9.1: Interview experience model + search
10. Task 10.1: Mock interview Agent
11. Task 11.1 → 11.2 → 11.3: Frontend pages (dashboard → positions → resumes/interviews)
12. Task 12.1: Deployment config

**After each task: run tests and commit.**
**After each phase: run full test suite.**
