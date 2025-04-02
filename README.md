# 369_quiz-CTk-
import customtkinter as ctk
import pandas as pd
import random
import pygame
import PIL

ctk.set_appearance_mode("default")
ctk.set_default_color_theme("369_theme.json")

dataset = pd.read_json("quiz_data.json")
dataset = dataset.sample(frac=1).reset_index(drop=True)
message_data = pd.read_json("messages.json")


class QuizApp(ctk.CTk):
    def __init__(self):
        # BG music
        pygame.mixer.init()
        pygame.mixer.music.load("QUIZ.mp3")
        pygame.mixer.music.set_volume(0.2)
        pygame.mixer.music.play(-1)

        super().__init__()
        self.title("369 Quiz")
        self.geometry("600x500")

        self.score = 0
        self.q_index = 0

        self.appLabel = ctk.CTkLabel(self, text="369 Quiz")
        self.appLabel.pack(pady=30)

        self.question_label = ctk.CTkLabel(self, text="")
        self.question_label.pack(pady=40)

        self.buttons = []
        for i in range(4):
            btn = ctk.CTkButton(self, text="", command=lambda idx=i: self.check_answer(idx))
            btn.pack(pady=15)
            self.buttons.append(btn)

        self.cor_lbl = ctk.CTkLabel(self, text="", fg_color="#008000", corner_radius=5,
                                    text_color="#FFEA00", pady=2, padx=4)
        self.next_btn = ctk.CTkButton(self, text="Next question", command=self.next_question)

        self.next_question()

    def next_question(self):
        if self.q_index < len(dataset):
            self.cor_lbl.pack_forget()  # Hide correct answer label when moving to next
            q = dataset.iloc[self.q_index]
            self.question_label.configure(text=q["question"])
            for i, opt in enumerate(q["options"]):
                self.buttons[i].configure(text=opt)
            self.next_btn.pack_forget()  # Hide next button until answer is selected
        else:
            self.show_result()

    def check_answer(self, idx):
        correct = dataset.iloc[self.q_index]["answer"]
        if self.buttons[idx].cget("text") == correct:
            self.score += 1
            pygame.mixer.Sound("ding.mp3").play()
            self.cor_lbl.pack_forget()  # No correct label needed
        else:
            pygame.mixer.Sound("buzz.mp3").play()
            self.cor_lbl.configure(text=f"Correct answer: {correct}")
            self.cor_lbl.pack(pady=10)

        self.q_index += 1
        self.next_btn.pack(pady=30)

    def show_result(self):
        for widget in self.winfo_children():
            widget.destroy()

        if self.score == len(dataset):
            level = "perfect"
        elif self.score >= len(dataset) * 0.75:
            level = "high"
        elif self.score >= len(dataset) * 0.4:
            level = "medium"
        else:
            level = "low"

        pygame.mixer.Sound("tada.mp3").play()
        msg = random.choice(message_data[level])

        result_label = ctk.CTkLabel(self, text=f"Your Score: {self.score}/{len(dataset)}\n\n{msg}",
                                    font=("Futura", 18), justify="center", wraplength=500)
        result_label.pack(pady=50)

        restart_btn = ctk.CTkButton(self, text="Try Again!", command=self.restart)
        restart_btn.pack(pady=30)

    def restart(self):
        self.score = 0
        self.q_index = 0

        for widget in self.winfo_children():
            widget.destroy()

        self.appLabel = ctk.CTkLabel(self, text="369 Quiz")
        self.appLabel.pack(pady=30)

        self.question_label = ctk.CTkLabel(self, text="")
        self.question_label.pack(pady=40)

        self.buttons = []
        for i in range(4):
            btn = ctk.CTkButton(self, text="", command=lambda idx=i: self.check_answer(idx))
            btn.pack(pady=15)
            self.buttons.append(btn)

        self.cor_lbl = ctk.CTkLabel(self, text="", fg_color="#008000", corner_radius=5,
                                    text_color="#FFEA00", pady=2, padx=4)
        self.next_btn = ctk.CTkButton(self, text="Next question", command=self.next_question)

        self.next_question()


if __name__ == "__main__":
    app = QuizApp()
    app.mainloop()
