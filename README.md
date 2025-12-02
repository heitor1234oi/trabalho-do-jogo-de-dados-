**# trabalho-do-jogo-de-dados-
Sistema simples de Jogo de Dados desenvolvido em Java com conceitos de POO. O jogador se cadastra, lança dois dados e o sistema calcula o resultado conforme as regras definidas, exibindo vitória ou derrota e permitindo jogar novamente.**# Sistema de Jogo de Dados

Autor: Heitor – 1F24

## Descrição do Projeto
Este projeto implementa um jogo simples de dados utilizando conceitos de Programação Orientada a Objetos. O jogador lança dois dados e o sistema informa o valor de cada dado, a soma e o resultado (vitória ou derrota) conforme a regra estabelecida.

##Levantamento de Requisitos

- O sistema deve permitir que o jogador lance dois dados.
- Cada dado deve gerar um valor aleatório entre 1 e 6.
- O sistema deve exibir os valores gerados pelos dados.
- O sistema deve calcular e exibir a soma dos dois valores.
- O jogador vence se a soma for maior ou igual a 8.
- O sistema deve permitir jogar novamente.

## 📘 Diagrama de Classes
<img width="1024" height="1024" alt="diagrama de classes" src="https://github.com/user-attachments/assets/da6edfb3-d382-4d94-9223-777053ded04e" />

### Detalhes das Classes

| Classe: Dado,Jogo
| Atributos :- valor: int ,dado1: Dado<br>, dado2: Dado
| Métodos :+ rolar(): int, + jogar(): void<br>+ verificarResultado(): String

##Diagrama de Caso de Uso
<img width="1024" height="1536" alt="Diagrama de Caso de Uso" src="https://github.com/user-attachments/assets/cf321235-3e94-452d-996d-2342b9988885" />



### Ator
- Jogador

### Caso de Uso
- Jogar Dados

com o diagrama de classese O código funciona em 4 partes principais que trabalham juntas:

Modelos (Modelos/Dados): São as informações básicas (como Jogador e Rodada).

Repositórios (Acesso ao Banco): Fazem o CRUD (Criar, Ler, Atualizar, Deletar) de forma automática.

Serviços (Cérebro/Regras): Decidem o que pode e o que não pode. Aqui se verifica se você é o dono da aposta e se rola o dado para finalizar.

Controladores (Telhado/Comandos): Recebem os comandos do navegador e dizem ao Serviço o que fazer, exibindo o resultado nas telas (Thymeleaf).

Em essência: O Navegador fala com o Controlador, que pergunta ao Serviço (Cérebro), que usa o Repositório para acessar os Modelos no banco de dados.

primeira parte Modelo
package com.exemplo.dicegame.model;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import jakarta.validation.constraints.NotNull;
import jakarta.validation.constraints.Size;
import java.io.Serializable;
import java.time.LocalDateTime;

@Entity
@Table(name = "rodada")
public class Rodada implements Serializable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank(message = "O Título da Rodada é obrigatório.")
    @Size(max = 60, message = "O Título deve ter no máximo 60 caracteres.")
    @Column(nullable = false, length = 60)
    private String tituloRodada;

    @NotNull(message = "O Valor da Aposta é obrigatório.")
    @Column(nullable = false)
    private Double valorAposta; // Novo campo: Valor apostado

    @Column(nullable = true)
    private Integer resultadoDado; // O resultado do dado (1 a 6)

    @Column(nullable = false)
    private Boolean finalizada = false; // Status da rodada

    @Column(nullable = false)
    private LocalDateTime dataCriacao = LocalDateTime.now();

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "jogador_id", nullable = false)
    private Jogador jogador; // Dono da Rodada (Aposta)

    // Construtores, Getters e Setters (Essenciais para o JPA/Thymeleaf)
    
    // Getters e Setters omitidos para brevidade, mas devem ser incluídos.
    // Exemplo:
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getTituloRodada() { return tituloRodada; }
    public void setTituloRodada(String tituloRodada) { this.tituloRodada = tituloRodada; }
    public Integer getResultadoDado() { return resultadoDado; }
    public void setResultadoDado(Integer resultadoDado) { this.resultadoDado = resultadoDado; }
    public Boolean getFinalizada() { return finalizada; }
    public void setFinalizada(Boolean finalizada) { this.finalizada = finalizada; }
    public Double getValorAposta() { return valorAposta; }
    public void setValorAposta(Double valorAposta) { this.valorAposta = valorAposta; }
    public Jogador getJogador() { return jogador; }
    public void setJogador(Jogador jogador) { this.jogador = jogador; }
    public LocalDateTime getDataCriacao() { return dataCriacao; }
    public void setDataCriacao(LocalDateTime dataCriacao) { this.dataCriacao = dataCriacao; }
}

Serviço:
package com.exemplo.dicegame.service;

import com.exemplo.dicegame.model.Jogador;
import com.exemplo.dicegame.model.Rodada;
import com.exemplo.dicegame.repository.RodadaRepository;
import jakarta.transaction.Transactional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;
import java.util.Random;

@Service
@Transactional
public class RodadaService {

    @Autowired
    private RodadaRepository rodadaRepository;

    @Autowired
    private JogadorService jogadorService; // Para buscar o Jogador logado

    public Rodada salvar(Rodada rodada, String emailDono) {
        // Encontra o jogador pelo email para definir o dono da aposta
        Jogador dono = jogadorService.buscarPorEmail(emailDono)
            .orElseThrow(() -> new UsernameNotFoundException("Jogador não encontrado."));
        
        rodada.setJogador(dono);
        rodada.setFinalizada(false); // Garante que a rodada começa não finalizada
        rodada.setResultadoDado(null); // O resultado só é gerado ao finalizar
        
        return rodadaRepository.save(rodada);
    }
    
    public Rodada atualizar(Rodada rodada, Long idRodada, String emailDono) {
        Rodada existente = buscarPorId(idRodada)
            .orElseThrow(() -> new IllegalArgumentException("Rodada não encontrada."));
        
        // Verifica se o usuário logado é o dono
        if (!existente.getJogador().getEmailAcesso().equals(emailDono)) {
            throw new SecurityException("Você não tem permissão para editar esta rodada.");
        }
        
        // Apenas permite a edição se a rodada não estiver finalizada
        if (existente.getFinalizada()) {
            throw new IllegalStateException("Rodada já finalizada e não pode ser editada.");
        }

        existente.setTituloRodada(rodada.getTituloRodada());
        existente.setValorAposta(rodada.getValorAposta());

        return rodadaRepository.save(existente);
    }

    public void remover(Long idRodada, String emailDono) {
        Rodada rodada = buscarPorId(idRodada)
            .orElseThrow(() -> new IllegalArgumentException("Rodada não encontrada."));

        // Verifica se o usuário logado é o dono
        if (!rodada.getJogador().getEmailAcesso().equals(emailDono)) {
            throw new SecurityException("Você não tem permissão para deletar esta rodada.");
        }
        
        rodadaRepository.deleteById(idRodada);
    }

    /**
     * Lógica de aposta e determinação do vencedor.
     * Apenas o dono pode clicar no botão para finalizar.
     */
    public Rodada finalizarRodada(Long idRodada, String emailDono) {
        Rodada rodada = buscarPorId(idRodada)
            .orElseThrow(() -> new IllegalArgumentException("Rodada não encontrada."));

        // Verifica se o usuário logado é o dono
        if (!rodada.getJogador().getEmailAcesso().equals(emailDono)) {
            throw new SecurityException("Você não tem permissão para finalizar esta rodada.");
        }
        
        if (rodada.getFinalizada()) {
             throw new IllegalStateException("Rodada já foi finalizada.");
        }

        // 1. Rola o dado (1 a 6)
        Random random = new Random();
        int resultado = random.nextInt(6) + 1; 

        // 2. Atualiza status e resultado
        rodada.setResultadoDado(resultado);
        rodada.setFinalizada(true);

        // 3. Persiste no banco
        return rodadaRepository.save(rodada);
    }

    @Transactional(readOnly = true)
    public Optional<Rodada> buscarPorId(Long id) {
        return rodadaRepository.findById(id);
    }

    @Transactional(readOnly = true)
    public List<Rodada> buscarTodas() {
        return rodadaRepository.findAll();
    }
}

Controller

package com.exemplo.dicegame.controller;

import com.exemplo.dicegame.model.Jogador;
import com.exemplo.dicegame.model.Rodada;
import com.exemplo.dicegame.service.JogadorService;
import com.exemplo.dicegame.service.RodadaService;
import jakarta.validation.Valid;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.stereotype.Controller;
import org.springframework.ui.ModelMap;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import java.util.List;
import java.util.Optional;

@Controller
@RequestMapping("/rodadas") // Novo caminho base para Rodadas
public class RodadaController {

    @Autowired
    private RodadaService rodadaService;

    @Autowired
    private JogadorService jogadorService; 
    
    // Método auxiliar para obter o email do usuário logado
    private String getUsuarioLogadoEmail() {
        return SecurityContextHolder.getContext().getAuthentication().getName();
    }

    @GetMapping("/index")
    public String listar(ModelMap model) {
        model.addAttribute("rodadas", rodadaService.buscarTodas());
        model.addAttribute("usuarioLogadoEmail", getUsuarioLogadoEmail());
        return "rodadas/index"; 
    }

    @GetMapping("/nova")
    public String preSalvar(Rodada rodada, ModelMap model) {
        // Envia a lista de jogadores (opcional, mas bom para formulário de criação se necessário)
        List<Jogador> jogadores = jogadorService.buscarTodos(); 
        model.addAttribute("jogadores", jogadores);
        return "rodadas/nova";
    }

    @PostMapping("/criar")
    public String salvar(@Valid Rodada rodada, BindingResult result, RedirectAttributes attr, ModelMap model) {
        if (result.hasErrors()) {
            List<Jogador> jogadores = jogadorService.buscarTodos();
            model.addAttribute("jogadores", jogadores);
            return "rodadas/nova";
        }
        
        rodadaService.salvar(rodada, getUsuarioLogadoEmail());
        attr.addFlashAttribute("sucesso", "Rodada criada com sucesso!");
        return "redirect:/rodadas/index";
    }

    @GetMapping("/editar/{id}")
    public String preEditar(@PathVariable("id") Long id, ModelMap model, RedirectAttributes attr) {
        Optional<Rodada> rodadaOpt = rodadaService.buscarPorId(id);
        if (rodadaOpt.isEmpty()) {
            attr.addFlashAttribute("erro", "Rodada não encontrada.");
            return "redirect:/rodadas/index";
        }
        
        Rodada rodada = rodadaOpt.get();
        // Verifica se o usuário logado é o dono antes de carregar o formulário
        if (!rodada.getJogador().getEmailAcesso().equals(getUsuarioLogadoEmail())) {
             attr.addFlashAttribute("erro", "Você não tem permissão para editar esta rodada.");
             return "redirect:/rodadas/index";
        }
        
        model.addAttribute("rodada", rodada);
        model.addAttribute("jogadores", jogadorService.buscarTodos());
        return "rodadas/editar";
    }

    @PutMapping("/atualizar/{id}")
    public String atualizar(@PathVariable("id") Long id, @Valid Rodada rodada, BindingResult result, RedirectAttributes attr, ModelMap model) {
        if (result.hasErrors()) {
            model.addAttribute("jogadores", jogadorService.buscarTodos());
            return "rodadas/editar";
        }

        try {
            rodadaService.atualizar(rodada, id, getUsuarioLogadoEmail());
            attr.addFlashAttribute("sucesso", "Rodada atualizada com sucesso!");
        } catch (SecurityException | IllegalStateException e) {
             attr.addFlashAttribute("erro", e.getMessage());
             return "redirect:/rodadas/index";
        }
        return "redirect:/rodadas/index";
    }

    @DeleteMapping("/excluir/{id}")
    public String excluir(@PathVariable("id") Long id, RedirectAttributes attr) {
        try {
            rodadaService.remover(id, getUsuarioLogadoEmail());
            attr.addFlashAttribute("sucesso", "Rodada removida com sucesso!");
        } catch (SecurityException e) {
            attr.addFlashAttribute("erro", e.getMessage());
        } catch (IllegalArgumentException e) {
             attr.addFlashAttribute("erro", "Rodada não encontrada.");
        }
        return "redirect:/rodadas/index";
    }
    
    // Novo Endpoint: Finalizar aposta e gerar o resultado
    @PostMapping("/finalizar/{id}")
    public String finalizarRodada(@PathVariable("id") Long id, RedirectAttributes attr) {
        try {
            Rodada rodadaFinalizada = rodadaService.finalizarRodada(id, getUsuarioLogadoEmail());
            attr.addFlashAttribute("sucesso", 
                "Rodada Finalizada! O resultado do dado foi: " + rodadaFinalizada.getResultadoDado());
        } catch (SecurityException | IllegalStateException e) {
            attr.addFlashAttribute("erro", e.getMessage());
        } catch (IllegalArgumentException e) {
            attr.addFlashAttribute("erro", "Rodada não encontrada.");
        }
        return "redirect:/rodadas/index";
    }
}


## Repositório no GitHub
https://github.com/heitor1234oi/trabalho-do-jogo-de-dados-
[Jogo_de_Dados_Heitor_1F24.pdf](https://github.com/user-attachments/files/23200703/Jogo_de_Dados_Heitor_1F24.pdf)

