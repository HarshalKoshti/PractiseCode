# PractiseCode
Basic practise code for new developer 
public ActionResult AddUser(NewUser usd)
        {
            dc.NewUsers.Add(usd);
            dc.SaveChanges();
            string message = string.Empty;
            switch(usd.UserId)
            {
                case 1:
                    message = "User already Exist.\\n please select different username";
                       break;
                case 2:
                    message = "Email address Already exist\\n Select different";
                    break;
                default:
                    message = "Registration successful" + usd.UserId.ToString();
                    SendActivationEmail(usd);
                    break;
            }
            ViewBag.Message = message;
            return View("UserDetails");
        }

        public ActionResult Activation()
        {
            ViewBag.Message = "Invalid Activation code.";
            if (RouteData.Values["id"] != null)
            {
                Guid activationCode = new Guid(RouteData.Values["id"].ToString());

                NewUser userActivation = dc.NewUsers.Where(p => p.IsVerified = activationCode).FirstOrDefault();

                if (userActivation != null)
                {
                    dc.NewUsers.Remove(userActivation);
                    dc.SaveChanges();
                    ViewBag.Message = "Activation successful.";
                }
            }

            return View();
        }

        private void SendActivationEmail(NewUser user)
        {
            Guid activationCode = new Guid();

            dc.NewUsers.Add(new NewUser
            {
                UserId = user.UserId,
                IsVerified = activationCode
            });
            dc.SaveChanges();

            using (MailMessage mm = new MailMessage("sender@gmail.com", user.Email))
            {
                mm.Subject = "Account Activation";
                string body = "Hello " + user.FirstName + ",";
                body += "<br /><br />Please click the following link to activate your account";
                body += "<br /><a href = '" + string.Format("{0}://{1}/Default/Activation/{2}", Request.Url.Scheme, Request.Url.Authority, activationCode) + "'>Click here to activate your account.</a>";
                body += "<br /><br />Thanks";
                mm.Body = body;
                mm.IsBodyHtml = true;
                SmtpClient smtp = new SmtpClient();
                smtp.Host = "smtp.gmail.com";
                smtp.EnableSsl = true;
                NetworkCredential NetworkCred = new NetworkCredential("sender@gmail.com", "<password>");
                smtp.UseDefaultCredentials = true;
                smtp.Credentials = NetworkCred;
                smtp.Port = 587;
                smtp.Send(mm);
            }
        }
