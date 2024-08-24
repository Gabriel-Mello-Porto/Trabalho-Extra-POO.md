# Main 

```java
public class Main {
    public static void main(String[] args) throws Exception {
        new InterfaceGraficaPatio();
    }
}
```

# InterfaceGraficaPatio

```java
import java.awt.BorderLayout;
import java.awt.Dimension;
import java.awt.GridLayout;
import java.awt.Image;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Random;
import javax.swing.ImageIcon;
import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JLabel;
import javax.swing.JOptionPane;
import javax.swing.JPanel;
import javax.swing.SpringLayout;
import java.util.ArrayList;

public class InterfaceGraficaPatio implements ActionListener {

    private ArrayList<Caminhao> vetorCaminhoes;
    
    // Variaveis globais
    JButton[] botoes = new JButton[6];
    JButton botaoComprar;
    JButton botaoVender;
    JButton botaoImprimir;
    JButton botaoSair;

    JFrame janelaPrincipal;

    ImageIcon cegonhaIcon;
    ImageIcon plataformaIcon;
    ArrayList<Integer> posicoesSendoUsadas = new ArrayList<Integer>();
    Random aleatorio = new Random();
    int idAtual = 0;


    public InterfaceGraficaPatio() {

        this.vetorCaminhoes = new ArrayList<Caminhao>();

        // JFrame princiapel (maior)
        janelaPrincipal = new JFrame("Revenda de Caminhoes");
        janelaPrincipal.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        janelaPrincipal.setSize(1024, 770);
        janelaPrincipal.setLocationRelativeTo(null);
        janelaPrincipal.setFocusable(true);

        // Imagens 
        String caminhoImagem = "imgs/FundoPatio.jpeg";
        cegonhaIcon = new ImageIcon("imgs/Cegonha.png");
        plataformaIcon = new ImageIcon("imgs/Plataforma.png");
        
        int escalaX = 140;
        int escalaY = 140;
        cegonhaIcon = scaleImageIcon(cegonhaIcon, escalaX, escalaY);
        plataformaIcon = scaleImageIcon(plataformaIcon, escalaX, escalaY);


        JPanel painelPrincipal = new JPanel(new BorderLayout());

        // JLabel de fundo
        JLabel fundo = new JLabel();
        ImageIcon imagemFundo = new ImageIcon(caminhoImagem);
        Image image = imagemFundo.getImage(); 
        Image scaledImage = image.getScaledInstance(1024, 1024, Image.SCALE_SMOOTH); // Dimensoes
        imagemFundo = new ImageIcon(scaledImage); 
        fundo.setIcon(imagemFundo);

        fundo.setLayout(new SpringLayout());

        // Botoes do patio
        JPanel gridBotoes = new JPanel();
        gridBotoes.setLayout(new GridLayout(2, 3));
        gridBotoes.setOpaque(false);

        for (int i = 0; i < 6; i++) {
            botoes[i] = new JButton();
            botoes[i].addActionListener(this);
            botoes[i].setPreferredSize(new Dimension(180, 110));
            gridBotoes.add(botoes[i]);

            // Deixa os botoes invisiveis (para colocar uma imagem de fundo)
            botoes[i].setOpaque(false);
            botoes[i].setContentAreaFilled(false);
            botoes[i].setBorderPainted(true);
        }

        // Caracteristica do SpringLayout
        SpringLayout layout = (SpringLayout) fundo.getLayout();
        fundo.add(gridBotoes);
        layout.putConstraint(SpringLayout.HORIZONTAL_CENTER, gridBotoes, 0, SpringLayout.HORIZONTAL_CENTER, fundo);
        layout.putConstraint(SpringLayout.VERTICAL_CENTER, gridBotoes, -100, SpringLayout.VERTICAL_CENTER, fundo);

        // Botoes inferiores
        JPanel painelBotoesInferiores = new JPanel(new GridLayout(1, 4));

        botaoComprar = new JButton("Comprar");
        botaoVender = new JButton("Vender");
        botaoImprimir = new JButton("Imprimir");
        botaoSair = new JButton("Sair");

        botaoComprar.addActionListener(this);
        botaoVender.addActionListener(this);
        botaoImprimir.addActionListener(this);
        botaoSair.addActionListener(this);

        painelBotoesInferiores.add(botaoComprar);
        painelBotoesInferiores.add(botaoVender);
        painelBotoesInferiores.add(botaoImprimir);
        painelBotoesInferiores.add(botaoSair);

        botaoComprar.setPreferredSize(new Dimension(120, 75));
        botaoVender.setPreferredSize(new Dimension(120, 75));
        botaoImprimir.setPreferredSize(new Dimension(120, 75));
        botaoSair.setPreferredSize(new Dimension(120, 75));


        // Adiciona tudo ao fundo
        painelPrincipal.add(fundo, BorderLayout.CENTER);
        painelPrincipal.add(painelBotoesInferiores, BorderLayout.SOUTH);
        janelaPrincipal.add(painelPrincipal);

        janelaPrincipal.setVisible(true);
    }

    @Override
    public void actionPerformed(ActionEvent e) {

        if (e.getSource() == botaoSair) {
            JOptionPane.showMessageDialog(janelaPrincipal , "Saindo");
            janelaPrincipal.dispose();
        }

        else if (e.getSource() == botaoComprar) {
            if (vetorCaminhoes.size() > 4) {
                JOptionPane.showMessageDialog(janelaPrincipal , "Patio atualmente cheio");
                throw new PatioCheioException("Patio atualmente cheio!\n");  
            }     
            comprarCaminhao();
        }

        else if (e.getSource() == botaoVender) {
            if (vetorCaminhoes.size() <= 0) {
                JOptionPane.showMessageDialog(janelaPrincipal , "Patio atualmente vazio");
                throw new PatioVazioException("Patio atualmente vazio!\n");
            } 

            try {
                int idAtual = Integer.parseInt(JOptionPane.showInputDialog(janelaPrincipal, "Informe o ID do caminhao que deseja vender"));
                venderCaminhao(idAtual);
            } catch (NumberFormatException ex) {
                JOptionPane.showMessageDialog(janelaPrincipal, "Id invalido");
            }
        }

        else if (e.getSource() == botaoImprimir) {
            imprimirCaminhoes();
        }

        // Qualquer botao do grid fora o 4 (que seria a entrada do patio)
        for (int i = 0; i < botoes.length; i++) {
            if (e.getSource() == botoes[i]) {
                imprimirDadosAtuais(i);
                return;
            }
        }
    }

    private void comprarCaminhao() {

        boolean tipoValido = false;
        int tipo;
        int cargaMax = 0;
        double tamCarroceria = 0;


        // Pega os dados
        do{
            tipo = Integer.parseInt(JOptionPane.showInputDialog(janelaPrincipal, "Escolha o tipo de caminhao:\n1) Cegonha\n2) Plataforma"));
            
            if (tipo == 1 || tipo == 2) {
                tipoValido = true;
            }

        } while (!tipoValido);
        
        String modelo = JOptionPane.showInputDialog(janelaPrincipal, "Modelo: (String)");
        int anoFabricacao = Integer.parseInt(JOptionPane.showInputDialog(janelaPrincipal, "Ano de fabricacao: (Int)"));
        double valor = Double.parseDouble(JOptionPane.showInputDialog(janelaPrincipal, "Valor: (Double)"));

        if (tipo == 1) {
            cargaMax = Integer.parseInt(JOptionPane.showInputDialog(janelaPrincipal, "Carga maxima: (Int)"));
        } else {
            tamCarroceria = Double.parseDouble(JOptionPane.showInputDialog(janelaPrincipal, "Tamanho da carroceria: (Double)"));
        }



        // Instancia a classe e adiciona em algum lugar vazio do patio
        if (tipo == 1) {
            Caminhao cegonha = new Cegonha(modelo, anoFabricacao, valor, cargaMax);
            
            int posicao;
            do {
                posicao = aleatorio.nextInt(6); // Gera uma posicao aleatoria entre 0 e 5
            } while (posicaoNaoEstaLivre(posicao));

            // Atualiza os dados
            posicoesSendoUsadas.add(posicao);        // Adiciona a posicao ao vetor
            botoes[posicao].setIcon(cegonhaIcon);    // Adiciona o icone ao botao atual
            vetorCaminhoes.add(cegonha);             // Adiciona o caminhao ao vetor
            cegonha.setId(idAtual);                  // Atualiza o Id
            idAtual++;
            JOptionPane.showMessageDialog(janelaPrincipal , "Caminhao " + cegonha.getModelo() + " adicionado com sucesso");

        } else if (tipo == 2) {
            Caminhao plataforma = new Plataforma(modelo, anoFabricacao, valor, tamCarroceria);

            int posicao;
            do {
                posicao = aleatorio.nextInt(6); // Gera uma posicao aleatoria entre 0 e 5
            } while (posicaoNaoEstaLivre(posicao));

            // Atualiza os dados
            posicoesSendoUsadas.add(posicao);          // Adiciona a posicao ao vetor
            botoes[posicao].setIcon(plataformaIcon);   // Adiciona o icone ao botao atual
            vetorCaminhoes.add(plataforma);            // Adiciona o caminhao ao vetor
            plataforma.setId(idAtual);                 // Atualiza o Id
            idAtual++;
            JOptionPane.showMessageDialog(janelaPrincipal , "Caminhao " + plataforma.getModelo() + " adicionado com sucesso");
        }
    }

    public void venderCaminhao(int id) {

        for (int i = 0; i < vetorCaminhoes.size(); i++) {
        
            Caminhao caminhaoAtual = vetorCaminhoes.get(i);         // Pega o caminhao atual
            int idCaminhaoAtual = caminhaoAtual.getId();            // Pega o id do caminhao atual
            int indiceCaminhaoAtual = posicoesSendoUsadas.get(i);   // Pega a posicao do caminhao atual
            
            if(idCaminhaoAtual == id) {
                vetorCaminhoes.remove(caminhaoAtual);                               // Remove caminhao do vetor
                posicoesSendoUsadas.remove(Integer.valueOf(indiceCaminhaoAtual));   // Remove posicao do vetor
                botoes[indiceCaminhaoAtual].setIcon(null);              // Remove o icone do botao atual
                JOptionPane.showMessageDialog(janelaPrincipal , "Caminhao " + caminhaoAtual.getModelo() +  " removido com sucesso");
                return;
            }
        }
        JOptionPane.showMessageDialog(janelaPrincipal ,"Id nao encontrado");
    }

    // Imprime os dados do caminhao que foi clicado
    private void imprimirDadosAtuais(int indiceAtual) {

        if (posicoesSendoUsadas.contains(indiceAtual)) {
            int indiceCaminhaoAtual = posicoesSendoUsadas.indexOf(indiceAtual);  // Pega o indice do caminhao atual na lista
            Caminhao caminhaoAtual = vetorCaminhoes.get(indiceCaminhaoAtual); // Pega o mesmo caminhao que foi clicado
    
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.append(caminhaoAtual.toString()).append("\n");
            JOptionPane.showMessageDialog(janelaPrincipal, stringBuilder.toString(), "Voce clicou no seguinte caminhao", JOptionPane.INFORMATION_MESSAGE);

        } else if (indiceAtual == 4) {
            JOptionPane.showMessageDialog(janelaPrincipal , "Entrada do patio!");

        } else {
            JOptionPane.showMessageDialog(janelaPrincipal , "Lugar vago");
        }
        
    }

    // Imprime todos os dados de todos os caminhoes atualmente no patio
    private void imprimirCaminhoes() {
    
    StringBuilder stringBuilder = new StringBuilder();
    
        if (vetorCaminhoes.size() <= 0){
            JOptionPane.showMessageDialog(janelaPrincipal , "Patio atualmente vazio");

        } else {
            for (int i = 0; i < vetorCaminhoes.size(); i++) {
                Caminhao caminhaoAtual = vetorCaminhoes.get(i);
                stringBuilder.append(caminhaoAtual.toString()).append("\n");
            }
            JOptionPane.showMessageDialog(janelaPrincipal, stringBuilder.toString(), "CAMINHOES ATUALMENTE NO PATIO:", JOptionPane.INFORMATION_MESSAGE);
        }
    }

    private boolean posicaoNaoEstaLivre(int posicao){
        // Garante que os inimigos nao fiquem na mesma posicao que o heroi ou entre si, nem com as armdilhas
        if (posicoesSendoUsadas.contains(posicao) || posicao == 4) {
            return true;
        }return false;
    }

    // Escala imagem
    private ImageIcon scaleImageIcon(ImageIcon icon, int x, int y) {
        Image img = icon.getImage();
        Image scaledImg = img.getScaledInstance(x, y, Image.SCALE_SMOOTH);
        return new ImageIcon(scaledImg);
    }

    public ArrayList<Caminhao> getVetorCaminhoesss() {
        return vetorCaminhoes;
    }
}
```

# Caminhao

```java
public abstract class Caminhao {
    private String modelo;
    private int anoFabricacao;
    private double valor;
    private int id;

    Caminhao(String modelo, int anoFabricacao, double valor) {
        this.modelo = modelo;
        this.anoFabricacao = anoFabricacao;
        this.valor = valor;
    }

    public String getModelo() {     
        return this.modelo;
    }
    public void setModelo(String modelo) {
        this.modelo = modelo;
    }

    public int getAnoFabricacao() {     
        return this.anoFabricacao;
    }
    public void setAnoFabricacao(int anoFabricacao) {
        this.anoFabricacao = anoFabricacao;
    }

    public double getValor() {     
        return this.valor;
    }
    public void setValor(double valor) {
        this.valor = valor;
    }

    public int getId() {     
        return this.id;
    }
    public void setId(int id) {
        this.id = id;
    }
    
    public abstract void imprimirClasse();

}
```

# Cegonha

```java
public class Cegonha extends Caminhao {

    private int cargaMax;

    Cegonha(String modelo, int anoFabricacao, double valor, int cargaMax) {
        super(modelo, anoFabricacao, valor);
        this.cargaMax = cargaMax;
    }

    public int getCargaMax() {     
        return this.cargaMax;
    }
    public void setCargaMax(int cargaMax) {
        this.cargaMax = cargaMax;
    }

    @Override
    public String toString() {
        return "Modelo: " + this.getModelo() + "\nAno de Fabricacao: " + this.getAnoFabricacao() + "\nCarga maxima: " + this.getCargaMax() + "\nValor: " + this.getValor() + "\nID unico: " + this.getId() + "\n";
    }

    public void imprimirClasse() {
        System.out.println("Classe atual: Plataforma");
    }
} 
```

# Plataforma 

```java
public class Plataforma extends Caminhao {

    private double tamCarroceria;       

    Plataforma(String modelo, int anoFabricacao, double valor, double tamCarroceria) {
        super(modelo, anoFabricacao, valor);
        this.tamCarroceria =tamCarroceria;
    }

    public double getTamCarroceria() {     
        return this.tamCarroceria;
    }
    public void setTamCarroceria(double tamCarroceria) {
        this.tamCarroceria = tamCarroceria;
    }

    @Override
    public String toString() {
        return "Modelo: " + this.getModelo() + "\nAno de Fabricacao: " + this.getAnoFabricacao() + "\nTamanho da Carroceria: " + this.getTamCarroceria() + " m" + "\nValor: " + this.getValor() + "\nID unico: " + this.getId() + "\n";
    }

    public void imprimirClasse() {
        System.out.println("Classe atual: Plataforma");
    }
}
``` 

# PatioCheioException

```java
public class PatioCheioException extends RuntimeException {
    
    PatioCheioException(String message) {
        super(message);
    }
}
```

# PatioVazioException

```java
public class PatioVazioException extends RuntimeException {
    
    PatioVazioException(String message) {
        super(message);
    }
}
```