FROM ubuntu:latest

RUN apt-get update && \ 
	apt-get install -y python 

RUN mkdir /home/myapp
WORKDIR /home/myapp
COPY calc.py .

CMD ["python", "calc.py"]

