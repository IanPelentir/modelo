# modelo
estrutura geral do projeto
lib/
├── apiservico.dart
├── estado.dart
├── main.dart
├── modelo.dart

Arquivo: modelo.dart
Estrutura e Classes
Classe Filme: Representa a entidade Filme com suas propriedades.
Construtor: Utilizado para criar uma instância da classe Filme.
Propriedades:
id: Identificador único do filme.
titulo: Título do filme.
imagem: URL da imagem do poster do filme.
descricao: Descrição do filme.
dataLancamento: Data de lançamento do filme.
nota: Nota média do filme.

class Filme {
  final String id;
  final String titulo;
  final String imagem;
  final String descricao;
  final String dataLancamento;
  final double nota;

  Filme({
    required this.id,
    required this.titulo,
    required this.imagem,
    required this.descricao,
    required this.dataLancamento,
    required this.nota,
  });

  factory Filme.fromJson(Map<String, dynamic> json) {
    return Filme(
      id: json['id'].toString(),
      titulo: json['title'],
      imagem: 'https://image.tmdb.org/t/p/w500${json['poster_path']}',
      descricao: json['overview'],
      dataLancamento: json['release_date'],
      nota: (json['vote_average'] as num).toDouble(),
    );
  }
}

Arquivo: apiservico.dart
Serviço de API
Classe ApiService: Responsável por fazer requisições HTTP e obter dados da API.

Variáveis:
apiUrl: URL base da API do TMDb.
apiKey: Chave de API para autenticação.
Método fetchFilmes: Faz uma requisição GET à API do TMDb para obter a lista de filmes populares.
Condicional if (response.statusCode == 200): Verifica se a resposta da API foi bem-sucedida (código de status 200).
Conversão JSON: Converte a resposta JSON em uma lista de objetos Filme.

import 'dart:convert';
import 'package:http/http.dart' as http;
import 'modelo.dart';

class ApiService {
  final String apiUrl = 'https://api.themoviedb.org/3/movie/popular';
  final String apiKey = '4262037ac8584fec0a64fdf52e796413';

  Future<List<Filme>> fetchFilmes() async {
    final response = await http.get(Uri.parse('$apiUrl?api_key=$apiKey'));

    if (response.statusCode == 200) {
      List jsonResponse = json.decode(response.body)['results'];
      return jsonResponse.map((movie) => Filme.fromJson(movie)).toList();
    } else {
      throw Exception('Falha ao carregar filmes');
    }
  }
}

Arquivo: estado.dart
Gerenciamento de Estado
Classe FilmesProvider: Gerencia o estado dos filmes usando o pacote provider.

Variáveis:
_filmes: Lista de filmes carregados.
_isLoading: Indica se os dados estão sendo carregados.
Método loadFilmes: Faz a chamada ao ApiService para carregar a lista de filmes e atualiza o estado.
Condicional if (response.statusCode == 200): Verifica se a resposta da API foi bem-sucedida.

import 'package:flutter/material.dart';
import 'apiservico.dart';
import 'modelo.dart';

class FilmesProvider with ChangeNotifier {
  List<Filme> _filmes = [];
  bool _isLoading = false;

  List<Filme> get filmes => _filmes;
  bool get isLoading => _isLoading;

  final ApiService _apiService = ApiService();

  Future<void> loadFilmes() async {
    _isLoading = true;
    notifyListeners();

    try {
      _filmes = await _apiService.fetchFilmes();
    } catch (e) {
      // Tratar erro
    } finally {
      _isLoading = false;
      notifyListeners();
    }
  }
}

Arquivo: main.dart
Interface do Usuário e Navegação
Função Principal main: Inicializa o aplicativo.
Classe MyApp: Define o tema e a estrutura principal do aplicativo.
Classe FilmesScreen: Tela inicial que exibe a lista de filmes populares.
Widget Scaffold: Estrutura básica da tela com AppBar, body e FloatingActionButton.
Widget ListView.builder: Constrói dinamicamente a lista de filmes.
Widget Card: Cria cartões para cada filme.
Widget ListTile: Permite que o usuário clique no filme para ver mais detalhes.
Método Navigator.push: Navega para a tela de detalhes do filme.
Condicional filmesProvider.isLoading ?: Verifica se os dados estão sendo carregados e exibe um CircularProgressIndicator se necessário.
Classe FilmeDetalhesScreen: Tela que exibe os detalhes de um filme quando clicado.
Widget ListView: Organiza as informações do filme.
Widget Padding: Adiciona espaçamento ao redor dos elementos.

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'estado.dart';
import 'modelo.dart';

void main() {
  runApp(
    ChangeNotifierProvider(
      create: (context) => FilmesProvider(),
      child: MyApp(),
    ),
  );
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(
        primarySwatch: Colors.blue,
        visualDensity: VisualDensity.adaptivePlatformDensity,
      ),
      home: FilmesScreen(),
    );
  }
}

class FilmesScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final filmesProvider = Provider.of<FilmesProvider>(context);

    return Scaffold(
      appBar: AppBar(
        title: Text('Lista de Filmes Populares'),
      ),
      body: filmesProvider.isLoading
          ? Center(child: CircularProgressIndicator())
          : ListView.builder(
              padding: EdgeInsets.all(8.0),
              itemCount: filmesProvider.filmes.length,
              itemBuilder: (context, index) {
                Filme filme = filmesProvider.filmes[index];
                return Card(
                  elevation: 5,
                  margin: EdgeInsets.symmetric(vertical: 8.0),
                  child: ListTile(
                    leading: Image.network(filme.imagem, width: 50, fit: BoxFit.cover),
                    title: Text(filme.titulo),
                    subtitle: Text('Nota: ${filme.nota}'),
                    onTap: () {
                      Navigator.push(
                        context,
                        MaterialPageRoute(
                          builder: (context) => FilmeDetalhesScreen(filme: filme),
                        ),
                      );
                    },
                  ),
                );
              },
            ),
      floatingActionButton: FloatingActionButton(
        onPressed: () {
          filmesProvider.loadFilmes();
        },
        child: Icon(Icons.refresh),
      ),
    );
  }
}

class FilmeDetalhesScreen extends StatelessWidget {
  final Filme filme;

  FilmeDetalhesScreen({required this.filme});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(filme.titulo),
      ),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: ListView(
          children: <Widget>[
            Center(
              child: Image.network(filme.imagem),
            ),
            SizedBox(height: 16.0),
            Text(
              filme.titulo,
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 8.0),
            Text(
              'Data de Lançamento: ${filme.dataLancamento}',
              style: TextStyle(fontSize: 16),
            ),
            SizedBox(height: 8.0),
            Text(
              'Nota: ${filme.nota}',
              style: TextStyle(fontSize: 16),
            ),
            SizedBox(height: 16.0),
            Text(
              filme.descricao,
              style: TextStyle(fontSize: 16),
            ),
          ],
        ),
      ),
    );
  }
}
