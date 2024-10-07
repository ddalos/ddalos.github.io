# 환경 구성
<pre>
  pip install fastapi
  pip install uvicorn
  pip install hypercon
</pre>

'''python

from fastapi import FastAPI

app = FastAPI()

@app.get("/")

def root():
	return {"Hello": "World"}
 '''
