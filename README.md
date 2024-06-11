-BackEnd

____Emailservice-----



package com.digitalisation.ims.Service;

import jakarta.mail.MessagingException;
import jakarta.mail.internet.MimeMessage;
import lombok.RequiredArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.mail.javamail.JavaMailSender;
import org.springframework.mail.javamail.MimeMessageHelper;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class EmailService {
    @Value("${spring.mail.username}")
    private String fromEmail;
    private final JavaMailSender mailSender;
    private static final String EMAIL_SUBJECT = "Tache à faire";
    private static final String EMAIL_TEXT = "<h4>bonjour,vous avez un nouveau étude impact à traiter vous avez</h4>";

    public void sendEmail(String to) {
        try {
            MimeMessage mimeMessage = mailSender.createMimeMessage();
            MimeMessageHelper helper = new MimeMessageHelper(mimeMessage, "utf-8");
            helper.setText(EMAIL_TEXT, true);
            helper.setTo(to);
            helper.setSubject(EMAIL_SUBJECT);
            helper.setFrom(fromEmail);
            mailSender.send(mimeMessage);
        } catch (MessagingException e) {
            throw new RuntimeException("Failed to send email", e);
        }
    }
}


-----Email Controller-------


package com.digitalisation.ims.Controller;

import com.digitalisation.ims.Service.EmailService;
import lombok.RequiredArgsConstructor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/email")
@RequiredArgsConstructor
public class EmailController {

    private final EmailService emailService;
    private static final Logger logger = LoggerFactory.getLogger(EmailController.class);

    @PostMapping("/send")
    public ResponseEntity<String> sendEmail(@RequestParam String to) {
        try {
            new Thread(() -> emailService.sendEmail(to)).start();
            return ResponseEntity.ok("{\"message\": \"Email is being sent\"}");
        } catch (Exception e) {
            logger.error("Failed to send email", e);
            return ResponseEntity.status(500).body("{\"error\": \"Failed to send email: " + e.getMessage() + "\"}");
        }
    }
}



-Front

-----Emailservice-----

import { Injectable } from '@angular/core';
import { HttpClient, HttpParams } from '@angular/common/http';
import { Observable, catchError, throwError } from 'rxjs';
@Injectable({
  providedIn: 'root'
})
export class EmailService {
  private baseUrl = 'http://localhost:8002/email';

  constructor(private http: HttpClient) { }

  sendEmail(to: string): Observable<any> {
    const params = new HttpParams().set('to', to);
    return this.http.post(`${this.baseUrl}/send`, null, { params });
  }


}
-----Ts imsstatus sendmail-----

sendEmail(responsableName: string): void {
    const responsable = this.ResponsableEI.find(user => user.name === responsableName);
    const email = responsable ? responsable.email : null;

    if (email) {
      this.emailService.sendEmail(email).subscribe(
        response => {
          console.log('Email envoyé avec succès', response);
          alert('Email envoyé avec succès');
        },
        error => {
          console.error('Erreur lors de l\'envoi de l\'email', error);
          const errorMessage = error.error?.error || 'Unknown error';
          alert('Erreur lors de l\'envoi de l\'email: ' + errorMessage);
        }
      );
    } else {
      console.error('Destinataire non trouvé');
      alert('Destinataire non trouvé');
    }
  }
