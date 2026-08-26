# Flutter

Dicas e recomendações reçacionadas a flutter

## Exemplos de Código

### 1. Exemplo em Dart Puro
Aqui está uma função simples em Dart que valida se um e-mail é válido:

```dart
bool validarEmail(String email) {
  final RegExp regex = RegExp(r'^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}\$');
  return regex.hasMatch(email);
}

void main() {
  String meuEmail = "teste@alternatribe.com";
  print("O e-mail é válido? \${validarEmail(meuEmail)}");
}
```

### 2. Exemplo em Flutter
Aqui está a estrutura de um componente de botão customizado em Flutter:

```dart
import 'package:flutter/material.dart';

class BotaoCustomizado extends StatelessWidget {
  final String texto;
  final VoidCallback aoClicar;

  const BotaoCustomizado({
    super.key,
    required this.texto,
    required this.aoClicar,
  });

  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        backgroundColor: Colors.blue,
        padding: const EdgeInsets.symmetric(horizontal: 20, vertical: 12),
      ),
      onPressed: aoClicar,
      child: Text(
        texto,
        style: const TextStyle(fontSize: 16, color: Colors.white),
      ),
    );
  }
}
```