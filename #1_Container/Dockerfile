FROM ubuntu:latest

RUN apt-get update && \ 
	apt-get install -y python 

MKDIR /home/myapp
WORKDIR /home/myapp
COPY calc.py .

CMD ["python", "calc.py"]

