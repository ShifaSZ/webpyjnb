FROM python:latest

RUN pip3 install --upgrade pip && \
    pip3 install jupyter

RUN mkdir /root/.jupyter

COPY ./jupyter_notebook_config.py /root/.jupyter/.

WORKDIR /root

CMD jupyter notebook --allow-root

