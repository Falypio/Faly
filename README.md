using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading;
using System.Windows.Forms;
namespace WindowsFormsApplication1
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }
        /// <summary>
        /// 线程同步
        /// 1.保证子线程里面的function为最小"原子级"
        /// 2.线程的等待与唤醒
        /// PS:这里主要用到2
        /// </summary>
        public Thread thA,thB,thC;
        private static ManualResetEvent eventOver = new ManualResetEvent(false);
        private static ManualResetEvent eventWait = new ManualResetEvent(false);
        private static ManualResetEvent eventThree = new ManualResetEvent(false);
        private void button1_Click(object sender, EventArgs e)
        {
            CheckForIllegalCrossThreadCalls = false;
            thA = new Thread(new ThreadStart(printA));
            thB = new Thread(new ThreadStart(printB));
            thC = new Thread(new ThreadStart(printC));
            thA.Name = "线程A";
            thB.Name = "线程B";
            thC.Name = "线程C";
            thA.Start();
            thB.Start();
            thC.Start();
            thA.Join();
            thB.Join();
            thC.Join();
        }
        private static object m_monitorObject = new object();
        public void printA()
        {
            for (int i = 0; i < 1000; i++)
            {
                if (eventOver.WaitOne(100, false))//线程A处于阻塞状态,等待唤醒
                {
                    Monitor.Enter(m_monitorObject);//获得锁
                    this.lbxShow.Items.Add(Thread.CurrentThread.Name + i);
                    eventOver.Reset();//线程A阻塞
                    eventWait.Set();//唤醒线程B
                    Monitor.Exit(m_monitorObject);//释放锁
                }
            }
        }
        public void printB()
        {
            eventOver.Set();//先唤醒A
            for (int i = 0; i < 1000; i++)
            {
                if (eventWait.WaitOne(100, false))//线程B处于阻塞状态,等待唤醒
                {
                    Monitor.Enter(m_monitorObject);//获得锁
                    this.lbxShow.Items.Add(Thread.CurrentThread.Name + i);
                    eventWait.Reset();//线程B阻塞
                    eventThree.Set();//唤醒线程C
                    Monitor.Exit(m_monitorObject);//释放锁
                }
            }
        }

        public void printC()
        {
            for (int i = 0; i < 1000; i++)
            {
                if (eventThree.WaitOne(100, false))//线程C处于阻塞状态,等待唤醒
                {
                    Monitor.Enter(m_monitorObject);//获得锁
                    this.lbxShow.Items.Add(Thread.CurrentThread.Name + i);
                    eventThree.Reset();//线程C阻塞
                    eventOver.Set();//唤醒线程A
                    Monitor.Exit(m_monitorObject);//释放锁
                }
            }
        }
    }
}
