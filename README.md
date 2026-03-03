import java.io.*;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;
import java.util.*;

enum Categoria {
    FESTA,
    SHOW,
    EVENTO_ESPORTIVO,
    CULTURAL,
    EDUCACIONAL
}

class Usuario {
    private int id;
    private String nome;
    private String email;
    private String cidade;

    public Usuario(int id, String nome, String email, String cidade) {
        this.id = id;
        this.nome = nome;
        this.email = email;
        this.cidade = cidade;
    }

    public int getId() { return id; }
    public String getNome() { return nome; }

    @Override
    public String toString() {
        return "ID: " + id + " | Nome: " + nome + " | Email: " + email + " | Cidade: " + cidade;
    }
}

class Evento implements Comparable<Evento> {
    private String nome;
    private String endereco;
    private Categoria categoria;
    private LocalDateTime horario;
    private String descricao;
    private int duracaoHoras;
    private Set<Integer> participantesIds;

    public Evento(String nome, String endereco, Categoria categoria, LocalDateTime horario, String descricao) {
        this.nome = nome;
        this.endereco = endereco;
        this.categoria = categoria;
        this.horario = horario;
        this.descricao = descricao;
        this.duracaoHoras = 2;
        this.participantesIds = new HashSet<>();
    }

    public String getNome() { return nome; }
    public LocalDateTime getHorario() { return horario; }
    public Set<Integer> getParticipantesIds() { return participantesIds; }

    public void setDuracaoHoras(int duracaoHoras) {
        this.duracaoHoras = duracaoHoras;
    }

    public void adicionarParticipante(int usuarioId) {
        participantesIds.add(usuarioId);
    }

    public void removerParticipante(int usuarioId) {
        participantesIds.remove(usuarioId);
    }

    public boolean isOcorriendoAgora() {
        LocalDateTime agora = LocalDateTime.now();
        return agora.isAfter(horario) && agora.isBefore(horario.plusHours(duracaoHoras));
    }

    public boolean isPassado() {
        return LocalDateTime.now().isAfter(horario.plusHours(duracaoHoras));
    }

    @Override
    public int compareTo(Evento outro) {
        return this.horario.compareTo(outro.horario);
    }

    public String toFile() {
        DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
        return nome + ";" + endereco + ";" + categoria + ";" +
               horario.format(formatter) + ";" + descricao + ";" +
               duracaoHoras + ";" +
               participantesIds.toString().replace("[", "").replace("]", "");
    }

    public static Evento fromFile(String linha) {
        try {
            String[] parts = linha.split(";");
            DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;

            Evento e = new Evento(
                parts[0],
                parts[1],
                Categoria.valueOf(parts[2]),
                LocalDateTime.parse(parts[3], formatter),
                parts[4]
            );

            if (parts.length > 5)
                e.setDuracaoHoras(Integer.parseInt(parts[5]));

            if (parts.length > 6 && !parts[6].isEmpty()) {
                String[] ids = parts[6].split(",");
                for (String id : ids) {
                    e.adicionarParticipante(Integer.parseInt(id.trim()));
                }
            }

            return e;
        } catch (Exception e) {
            return null;
        }
    }

    @Override
    public String toString() {
        String status = "";
        if (isOcorriendoAgora()) status = " [OCORRENDO AGORA]";
        else if (isPassado()) status = " [PASSADO]";

        return "\n--- " + nome + status + " ---" +
               "\nCategoria: " + categoria +
               "\nHorário: " + horario.format(DateTimeFormatter.ofPattern("dd/MM/yyyy HH:mm")) +
               "\nEndereço: " + endereco +
               "\nDescrição: " + descricao +
               "\nParticipantes Confirmados: " + participantesIds.size();
    }
}

public class Main {

    static List<Evento> eventos = new ArrayList<>();
    static List<Usuario> usuarios = new ArrayList<>();
    static Scanner sc = new Scanner(System.in);
    static Usuario usuarioLogado = null;

    public static void main(String[] args) {

        carregarEventos();

        while (true) {

            System.out.println("\n========== SISTEMA DE EVENTOS ==========");

            if (usuarioLogado == null) {
                System.out.println("1 - Cadastrar Usuário");
                System.out.println("2 - Login");
                System.out.println("3 - Listar Eventos");
                System.out.println("4 - Sair");
            } else {
                System.out.println("Olá, " + usuarioLogado.getNome());
                System.out.println("1 - Cadastrar Evento");
                System.out.println("2 - Listar Eventos");
                System.out.println("3 - Participar de Evento");
                System.out.println("4 - Logout");
            }

            System.out.print("Escolha: ");
            int opcao = sc.nextInt();
            sc.nextLine();

            if (usuarioLogado == null) {
                switch (opcao) {
                    case 1 -> cadastrarUsuario();
                    case 2 -> login();
                    case 3 -> listarEventos();
                    case 4 -> {
                        salvarEventos();
                        System.out.println("Encerrando...");
                        System.exit(0);
                    }
                    default -> System.out.println("Opção inválida.");
                }
            } else {
                switch (opcao) {
                    case 1 -> cadastrarEvento();
                    case 2 -> listarEventos();
                    case 3 -> participarEvento();
                    case 4 -> usuarioLogado = null;
                    default -> System.out.println("Opção inválida.");
                }
            }
        }
    }

    static void cadastrarUsuario() {
        System.out.print("Nome: ");
        String nome = sc.nextLine();
        System.out.print("Email: ");
        String email = sc.nextLine();
        System.out.print("Cidade: ");
        String cidade = sc.nextLine();

        int id = usuarios.size() + 1;
        usuarios.add(new Usuario(id, nome, email, cidade));

        System.out.println("Usuário cadastrado! ID: " + id);
    }

    static void login() {
        if (usuarios.isEmpty()) {
            System.out.println("Nenhum usuário cadastrado.");
            return;
        }

        usuarios.forEach(System.out::println);

        System.out.print("Digite seu ID: ");
        int id = sc.nextInt();
        sc.nextLine();

        for (Usuario u : usuarios) {
            if (u.getId() == id) {
                usuarioLogado = u;
                System.out.println("Login realizado!");
                return;
            }
        }

        System.out.println("ID não encontrado.");
    }

    static void cadastrarEvento() {
        try {
            System.out.print("Nome: ");
            String nome = sc.nextLine();

            System.out.print("Endereço: ");
            String endereco = sc.nextLine();

            System.out.print("Categoria (FESTA, SHOW, EVENTO_ESPORTIVO, CULTURAL, EDUCACIONAL): ");
            Categoria categoria = Categoria.valueOf(sc.nextLine().toUpperCase());

            System.out.print("Data e hora (AAAA-MM-DDTHH:MM): ");
            LocalDateTime horario = LocalDateTime.parse(sc.nextLine());

            System.out.print("Descrição: ");
            String descricao = sc.nextLine();

            Evento e = new Evento(nome, endereco, categoria, horario, descricao);
            e.adicionarParticipante(usuarioLogado.getId());

            eventos.add(e);

            System.out.println("Evento cadastrado com sucesso!");

        } catch (Exception e) {
            System.out.println("Erro ao cadastrar evento. Verifique os dados.");
        }
    }

    static void listarEventos() {
        if (eventos.isEmpty()) {
            System.out.println("Nenhum evento cadastrado.");
            return;
        }

        Collections.sort(eventos);
        eventos.forEach(System.out::println);
    }

    static void participarEvento() {
        listarEventos();

        System.out.print("Digite o nome do evento: ");
        String nome = sc.nextLine();

        for (Evento e : eventos) {
            if (e.getNome().equalsIgnoreCase(nome)) {
                e.adicionarParticipante(usuarioLogado.getId());
                System.out.println("Participação confirmada!");
                return;
            }
        }

        System.out.println("Evento não encontrado.");
    }

    static void salvarEventos() {
        try (PrintWriter pw = new PrintWriter("eventos.txt")) {
            for (Evento e : eventos) {
                pw.println(e.toFile());
            }
        } catch (Exception e) {
            System.out.println("Erro ao salvar.");
        }
    }

    static void carregarEventos() {
        File file = new File("eventos.txt");
        if (!file.exists()) return;

        try (Scanner fileScanner = new Scanner(file)) {
            while (fileScanner.hasNextLine()) {
                Evento e = Evento.fromFile(fileScanner.nextLine());
                if (e != null) eventos.add(e);
            }
        } catch (Exception ignored) {}
    }
}
