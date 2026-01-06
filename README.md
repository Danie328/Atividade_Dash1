import pandas as pd
import plotly.express as px
from dash import Dash, dcc, html
from dash.dependencies import Input, Output
import numpy as np

df = pd.read_csv("C:/Users/PC/Desktop/M.python/ecommerce_estatistica (1).csv")

df=pd.DataFrame(df)
def cria_graficos(df):



    df['Qtd_Vendidos'] = df['Qtd_Vendidos'].astype(str).str.replace('+', '', regex=False).str.replace('mil', '000',
                                                                                                      regex=False).str.replace(
        'M', '000000', regex=False).str.strip()
    df['Qtd_Vendidos'] = pd.to_numeric(df['Qtd_Vendidos'], errors='coerce')


    df['Nota'] = pd.to_numeric(df['Nota'], errors='coerce')
    df = df.dropna(subset=['Nota', 'Qtd_Vendidos']).copy()

    # -------------------------------------------------------------
    # 2. CRIAÇÃO DOS GRÁFICOS
    # -------------------------------------------------------------

    """Gráfico de Histograma"""
    fig1 = px.histogram(df, x='Preço', nbins=40, title='1. Distribuição de Preços')

    """Gráfico de pizza"""
    df_genero_counts = df['Gênero'].value_counts().reset_index(name='Contagem')
    df_genero_counts.columns = ['Gênero', 'Contagem']
    fig2 = px.pie(df_genero_counts, values='Contagem', names='Gênero',
                  title='5. Gráfico de Pizza: Proporção de Produtos por Gênero', hole=.3)

    """Gráfico de Dispersão"""
    fig3 = px.scatter(df, x='Preço', y='Nota', color='Preço', hover_data=['Título'], size_max=60)
    fig3.update_layout(xaxis_title='Preço (R$)', yaxis_title='Nota Média',
                       title='2. Dispersão: Preço vs. Nota (Colorido por Preço)')

    """ Mapa de Calor. (df_heatmap criado aqui para resolver o TypeError)"""
    df_heatmap = df.groupby(['Gênero', 'Material'])['Nota'].mean().unstack()
    df_heatmap = df_heatmap.fillna(0)

    fig4 = px.imshow(
        df_heatmap, labels=dict(x="Material", y="Gênero", color="Média da Nota"),
        x=df_heatmap.columns, y=df_heatmap.index, color_continuous_scale="Viridis", title="3. Mapa de Calor: Média da Nota por Gênero e Material",
        aspect="auto")
    fig4.update_xaxes(side="top")

    """Gráfico de Barra (Top 10 Marcas)"""
    df_top10_marcas = df.groupby('Marca')['Qtd_Vendidos'].sum().nlargest(10).reset_index()
    fig5 = px.bar(df_top10_marcas.sort_values(by='Qtd_Vendidos', ascending=False),
                  x='Marca', y='Qtd_Vendidos', color='Qtd_Vendidos', barmode='group',
                  color_discrete_sequence=px.colors.qualitative.Bold, opacity=1)
    fig5.update_layout(
        title='4. Gráfico de Barra: Top 10 Marcas por Qtd. Vendida',
        xaxis_title=('Marca'),
        yaxis_title=('Total de unidades vendidos'))

    """Gráfico de Densidade"""
    fig6 = px.density_contour(df, x='Nota', y='Preço', marginal_x="box", marginal_y="violin",
                              title='6. Gráfico de Densidade 2D: Nota e Preço')
    fig6.update_layout(xaxis_title='Nota Média', yaxis_title='Preço (R$)')

    """Gráfico de Regressão"""
    fig7 = px.scatter(df, x='N_Avaliações', y='Qtd_Vendidos', trendline="ols",
                      title='7. Gráfico de Regressão: Avaliações vs. Vendas')
    fig7.update_layout(xaxis_title='Nº de Avaliações', yaxis_title='Quantidade Vendida')

    return fig1, fig2, fig3, fig4, fig5, fig6, fig7


def cria_app(df):
    app = Dash(__name__)

    fig1, fig2, fig3, fig4, fig5, fig6, fig7 = cria_graficos(df)

    app.layout = html.Div([
        dcc.Graph(figure=fig1),
        dcc.Graph(figure=fig2),
        dcc.Graph(figure=fig3),
        dcc.Graph(figure=fig4),
        dcc.Graph(figure=fig5),
        dcc.Graph(figure=fig6),
        dcc.Graph(figure=fig7)
    ])

    return app


if __name__ == '__main__':

    app = cria_app(df)


    app.run(debug=True, port=8050)return app


if __name__ == '__main__':
