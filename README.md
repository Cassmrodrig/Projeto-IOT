# Projeto-IOT

import requests
from bs4 import BeautifulSoup
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
import datetime

# Função para enviar e-mail
def send_email(body):
    # Configurações do servidor de e-mail
    sender_email = "seuemail@gmail.com"
    sender_password = "suasenha"
    receiver_email = "destinatario@example.com"

    # Construindo o e-mail
    message = MIMEMultipart()
    message["From"] = sender_email
    message["To"] = receiver_email
    message["Subject"] = "Chamados abertos há mais de três dias"

    # Adicionando o corpo do e-mail
    message.attach(MIMEText(body, "plain"))

    # Conexão com o servidor SMTP do Gmail
    server = smtplib.SMTP_SSL("smtp.gmail.com", 465)
    server.login(sender_email, sender_password)

    # Enviando o e-mail
    server.sendmail(sender_email, receiver_email, message.as_string())
    server.quit()

# URL do site
url = "https://liveops-americas.seidor.com/#/tickets-v2?Expanded=true"

# Dados de login
login_data = {
    "email": "cassy.rodrigues@seidor.com.br",
    "password": "Eoqu&1roz?"
}

# Fazendo login
with requests.Session() as session:
    session.post(url, data=login_data)
    
    # Obtendo a página após o login
    response = session.get(url)
    
    # Parseando a página com BeautifulSoup
    soup = BeautifulSoup(response.content, "html.parser")
    
    # Encontrando os chamados abertos em mais de três dias
    chamados = soup.find_all("div", class_="ticket")
    
    chamados_abertos = []
    
    for chamado in chamados:
        data_abertura_str = chamado.find("span", class_="data").text
        data_abertura = datetime.datetime.strptime(data_abertura_str, "%d/%m/%Y")
        dias_em_aberto = (datetime.datetime.now() - data_abertura).days
        
        if dias_em_aberto > 3:
            chamados_abertos.append(chamado.find("span", class_="id").text)
    
    # Verificando se há chamados abertos há mais de três dias
    if chamados_abertos:
        # Enviando e-mail com os chamados abertos há mais de três dias
        send_email("\n".join(chamados_abertos))
        print("E-mail enviado com sucesso!")
    else:
        print("Não há chamados abertos há mais de três dias.")
