import spacy
import pandas as pd
import unicodedata
import nltk

# Download dos recursos necessários do NLTK e SpaCy (executar uma vez)
nltk.download('punkt')
# Para SpaCy, baixe o modelo português na linha de comando: python -m spacy download pt_core_news_sm

# Função para remover acentos de uma string
def remover_acentos(texto):
    texto = unicodedata.normalize('NFD', texto)
    texto = ''.join(c for c in texto if unicodedata.category(c) != 'Mn')
    return texto

# Carrega modelo SpaCy para português
nlp = spacy.load("pt_core_news_sm")

# Exemplo de criação de um DataFrame Pandas com nomes próprios acentuados
# Pode-se carregar de arquivo CSV na prática
dados_nomes = {
    "nome": [
        "Maria", "João", "Páulu", "Albértu", "Ãna", "Rafaéu",
        "Fernãnda", "Lúcas", "Marcélu", "José", "Mariãna", "Carlos",
        "Beatríz", "Pêdru", "Joãna", "André", "Simõni", "Gabriéu",
        "Cárla", "Daniéu", " María di Fátima"
    ]
}
df_nomes = pd.DataFrame(dados_nomes)

# Criar dicionário para consulta rápida: chave = nome sem acento em minúsculas, valor = nome acentuado
dicionario_nomes = {
    remover_acentos(nome.lower()): nome for nome in df_nomes["nome"]
}

def recuperar_acentos(texto):
    doc = nlp(texto)
    tokens_corrigidos = []
    for token in doc:
        if token.ent_type_ == "PER":  # Entidade do tipo Pessoa
            chave = remover_acentos(token.text.lower())
            nome_corrigido = dicionario_nomes.get(chave, token.text)
            tokens_corrigidos.append(nome_corrigido)
        else:
            tokens_corrigidos.append(token.text)
    return " ".join(tokens_corrigidos)

if __name__ == "__main__":
    texto_digitado = "maria jose silva e joao paulo foram a sao paulo"
    texto_corrigido = recuperar_acentos(texto_digitado)
    print("Texto digitado:", texto_digitado)
    print("Texto com acentos recuperados:", texto_corrigido)
