#include <iostream>
#include <fstream>
#include <vector>
#include <bitset>
using namespace std;

// Convertit une chaîne en bits
string messageToBinary(const string& message) {
    string binary;
    for (char c : message) {
        binary += bitset<8>(c).to_string();
    }
    return binary;
}

// Convertit une chaîne binaire en texte
string binaryToMessage(const string& binary) {
    string message;
    for (size_t i = 0; i + 7 < binary.size(); i += 8) {
        string byteStr = binary.substr(i, 8);
        char c = static_cast<char>(bitset<8>(byteStr).to_ulong());
        if (c == '#') break;
        message += c;
    }
    return message;
}

// Fonction d'encodage
void encoderBMP(const string& inputFile, const string& outputFile, const string& message) {
    ifstream in(inputFile, ios::binary);
    if (!in) {
        cerr << "Erreur d'ouverture du fichier source.\n";
        return;
    }

    vector<unsigned char> data((istreambuf_iterator<char>(in)), {});
    in.close();

    // Récupère l'offset vers les données pixels (à l'octet 10)
    int offset = *reinterpret_cast<int*>(&data[10]);

    string binaryMessage = messageToBinary(message + "#");
    size_t msgLen = binaryMessage.size();

    if (offset + msgLen > data.size()) {
        cerr << "L'image est trop petite pour contenir ce message.\n";
        return;
    }

    for (size_t i = 0; i < msgLen; ++i) {
        data[offset + i] &= 0xFE;  // met le LSB à 0
        data[offset + i] |= (binaryMessage[i] - '0');  // insère le bit
    }

    ofstream out(outputFile, ios::binary);
    if (!out) {
        cerr << "Erreur d'écriture du fichier de sortie.\n";
        return;
    }

    out.write(reinterpret_cast<char*>(&data[0]), data.size());
    out.close();

    cout << "Message encodé avec succès dans '" << outputFile << "'.\n";
}

// Fonction de décodage
void decoderBMP(const string& inputFile) {
    ifstream in(inputFile, ios::binary);
    if (!in) {
        cerr << "Erreur d'ouverture du fichier.\n";
        return;
    }

    vector<unsigned char> data((istreambuf_iterator<char>(in)), {});
    in.close();

    int offset = *reinterpret_cast<int*>(&data[10]);
    string bits, message;

    for (size_t i = offset; i < data.size(); ++i) {
        bits += (data[i] & 1) ? '1' : '0';

        if (bits.size() >= 8 && bits.substr(bits.size() - 8) == "00100011")  // '#'
            break;
    }

    message = binaryToMessage(bits);
    cout << "Message décodé : " << message << "\n";
}
