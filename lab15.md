# Цель работы

Реализовать решение прикладной задачи из предметной области с использованием одного из идиоматических паттернов: Producer-Consumer.

# Ход работы

## Задание 1. Очередь обработки аудиофайлов

Главный поток отправляет задачи загрузки в канал, два воркера забирают их из общей очереди и обрабатывают. Завершение синхронизировано закрытием канала и `join()` потоков.

```rust
use std::sync::mpsc;
use std::thread;

struct Job {
    file: String,
}

fn main() {
    let (tx, rx) = mpsc::channel::<Job>();
    let rx = std::sync::Arc::new(std::sync::Mutex::new(rx));

    let mut workers = vec![];
    for id in 1..=2 {
        let rx = std::sync::Arc::clone(&rx);
        workers.push(thread::spawn(move || {
            loop {
                let job = rx.lock().unwrap().recv();
                match job {
                    Ok(Job { file }) => {
                        let status = if id == 1 { "done" } else { "done" };
                        println!("воркер {}: {} -> {}", id, file, status);
                    }
                    Err(_) => break,
                }
            }
        }));
    }
```

Демонстрация решения:

```rust
    for file in ["a.mp3", "b.wav", "c.mp3"] {
        tx.send(Job { file: file.into() }).unwrap();
    }
    drop(tx);

    for w in workers {
        w.join().unwrap();
    }
    println!("очередь пуста, все задачи обработаны");
}
```

![Рисунок 1](img/lab15-task1.png)

Рисунок 1 - Результат выполнения задания 1

Результат: три задачи обработаны двумя воркерами; порядок строк зависит от планировщика, все файлы получили статус done.

# Вывод

Прикладная задача решена с использованием перечисленных паттернов; корректность подтверждена выводом программ.
