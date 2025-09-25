import spacy
import pandas as pd
import unicodedata

# Função para remover acentos da palavra (normalização)
def remove_acentos(texto):
    texto = unicodedata.normalize('NFD', texto)
    texto = ''.join(c for c in texto if unicodedata.category(c) != 'Mn')
    return texto

# Carregar modelo SpaCy para NER
nlp = spacy.load("pt_core_news_sm")  # modelo em português

# Carregar banco de nomes próprios acentuados em CSV
# CSV esperado: coluna "nome" com nomes acentuados
nomes_acentuados_df = pd.read_csv("nomes_acentuados.csv")
# Criar dicionário para consulta sem acentos -> com acentos
dicionario_nomes = {remove_acentos(nome.lower()): nome for nome in nomes_acentuados_df["nome"]}

# Função para recuperar acentos de nomes próprios no texto
def recuperar_acentos(texto_sem_acentos):
    doc = nlp(texto_sem_acentos)
    tokens_corrigidos = []
    for token in doc:
        palavra = token.text
        if token.ent_type_ == "PER":  # se for nome próprio identificado
            chave = remove_acentos(palavra.lower())
            palavra_com_acentos = dicionario_nomes.get(chave, palavra)
            tokens_corrigidos.append(palavra_com_acentos)
        else:
            tokens_corrigidos.append(palavra)
    return " ".join(tokens_corrigidos)

# Exemplo de uso
if __name__ == "__main__":
    texto_entrada = "maria jose silva e joao paulo foram a sao paulo"
    texto_acentuado = recuperar_acentos(texto_entrada)
    print("Texto original:", texto_entrada)
    print("Texto com acentos recuperados:", texto_acentuado)
