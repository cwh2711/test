###builder##

# 拉官方映像檔
FROM python:3.11.4-slim-buster as builder

# 設定工作區域
WORKDIR /usr/src/app

# 設定映像檔
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# 下載編譯器
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc

# 下載偵錯程式
RUN pip install --upgrade pip
RUN pip install flake8==6.0.0
COPY . /usr/src/app/
RUN flake8 --ignore=E501,F401,W291,W293 .

# 使用wheel安裝requirements.txt內的內容並放進wheels目錄
COPY ./requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/app/wheels -r requirements.txt


###main##
# 拉官方映像檔
FROM python:3.11.4-slim-buster

# 創造新使用者app
RUN addgroup --system app && adduser --system --group app

# 創造使用者app的工作區域
RUN mkdir -p /home/app

# 創造目錄web作為工作區域、目錄staticfiles處理靜態資料、目錄mediafiles處理使用者上傳資料
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
RUN mkdir $APP_HOME/staticfiles
RUN mkdir $APP_HOME/mediafiles
WORKDIR $APP_HOME

# 安裝builder裡的內容
RUN apt-get update && apt-get install -y --no-install-recommends netcat
COPY --from=builder /usr/src/app/wheels /wheels
COPY --from=builder /usr/src/app/requirements.txt .
RUN pip install --upgrade pip
RUN pip install --no-cache /wheels/*

# 複製entrypoint.sh、給予執行權限
COPY ./entrypoint.sh .
RUN chmod +x  $APP_HOME/entrypoint.sh

# 複製目錄的資料進來
COPY . $APP_HOME

# 給使用者app權限
RUN chown -R app:app $APP_HOME

# 更改為使用者app
USER app

# 跑entrypoint.sh
ENTRYPOINT ["/home/app/web/entrypoint.sh"]
